---
name: xray
description: LinkedIn x-ray search. Find public LinkedIn profiles via site-restricted web search (site:linkedin.com/in/) without logging in. Boolean query builder, runs through a search API, a browser tool, or pasted results. Use for "x-ray search", "find profiles", "sourcing candidates", "recruitin", or boolean profile search.
---

# X-ray search

LinkedIn blocks login-based scraping and bot detection will kill an account. X-ray is the safe workaround: public LinkedIn profiles are indexed by search engines, so `site:linkedin.com/in/` queries find them **without ever touching a LinkedIn session**. No account, no kick-out risk.

## Capability: the agent cannot run web searches reliably on its own

Vanilla curl against Google, Bing, DuckDuckGo, Startpage, Mojeek and SearXNG is bot-blocked (tested: DDG serves a bot-check after one query, Bing returns zero organic results, Google blocks with 405). X-ray only works through one of these lanes:

- **API mode (recommended):** a search API with a key in the environment. Fully automatic, JSON results, no bot walls. Two viable providers:
  - `SERPER_API_KEY` (Google index, the deepest LinkedIn coverage, full boolean). ~$0.0005/query, 2,500 free on signup. **Preferred for x-ray.**
  - `BRAVE_API_KEY` (independent index, 2,000 free/month): good operators, slightly thinner LinkedIn coverage.

  Serper recipe (verified working):

  ```bash
  curl -s -X POST "https://google.serper.dev/search" \
    -H "X-API-KEY: $SERPER_API_KEY" -H "Content-Type: application/json" \
    -d '{"q":"<query>"}'
  ```

  Parse `organic[]`: each entry has `title`, `link`, `snippet`, so the profile list comes straight out of it. A full boolean query with `OR` chains returns 10 results per page; page with `&page=2`-style params or narrow the query for more.
- **Paste mode (default):** the user runs the query in their own browser (Google or Startpage, see engine comparison) or via recruitin.net, and pastes the results. The agent does query building + extraction.
- **Browser mode:** a browser automation tool runs the queries in a real browser.

Ask the user which lane is available before running queries. DuckDuckGo is **not** a usable x-ray lane: it has no `OR` operator, its own docs admit advanced-syntax bugs, and it bot-blocks curl.

## Query anatomy

X-ray queries are boolean: **who** + **where** + **site:** + **exclusions** + **credentials** + **company/industry**.

Example (the classic shape):

```
"sales manager" "munich" -intitle:"profiles" -inurl:"dir/"
site:linkedin.com/in/ OR site:linkedin.com/pub/
masters OR mba OR master OR diplome OR msc OR magister OR magisteres OR maitrise
"airbus"
```

Component by component:

| Component | Pattern | Why |
|---|---|---|
| Role | `"sales manager"` (exact phrase) | matches the headline/experience, not a mention |
| Location | `"munich"` (exact phrase) | profiles list location; use the city name, not "Germany" alone |
| Site | `site:linkedin.com/in/ OR site:linkedin.com/pub/` | restricts to public profiles; `/pub/` catches legacy profiles |
| Country | `site:de.linkedin.com/in/` (verified) | country subdomains isolate national profiles. German market: `de`, `at`, `ch`; also `uk`, `ie`, `au`, `ca`, `in` |
| Exclusions | `-intitle:"profiles" -inurl:"dir/"` | drops LinkedIn's own directory pages from results |
| Credentials | degree list: `masters OR mba OR master OR diplome OR msc OR magister OR magisteres OR maitrise` | filters seniority/school level (DE market: magister/diplome matter) |
| Company | `"airbus"` (exact phrase) | restricts to current/past company; swap per engagement |
| Proximity | `"sales manager" AROUND(3) "airbus"` (verified) | two terms within N words of each other; tighter than AND for profile snippets |

## Workflow

### 1. Build queries

From the engagement context (client, role, market; ideally the target list from the `competitor-research` skill), build one query per role × location combination. Always include site:, exclusions, and credentials. Add the company filter only when targeting one company.

Role terms: the exact role title plus its common variants (`"sales manager" OR "sales director" OR "vertriebsleiter"`; DE roles need the German titles too). Location terms: the city and its region (`"munich" OR "münchen" OR "oberbayern"`).

Completion: one complete query per combination; every query has role, location, site:, exclusions and credentials (company only when targeting one company).

### 2. Run (per lane)

- Paste mode: show the user the finished queries, one per line, ready to paste into Google/recruitin.net. Ask them to paste results back.
- API mode: run each query through the configured API, parse the organic results.
- Browser mode: run each query in the browser tool, extract result links + titles.

Completion: results collected for every query, or a query is explicitly dropped.

### 3. Extract profiles

From each result: name, profile URL, headline, location, current company, past companies (when visible). Keep a flat list and dedupe by profile URL. Keep the source query per profile (it tells you which role/location signal matched).

Completion: every result extracted; duplicates removed; the list shows name, URL, and the matching signal.

### 4. Deliver

Write the profile list to the engagement file (default `~/recruiting/<client-slug>/profiles.md`). When the goal is market or company discovery, also extract the **companies** from the profiles' past and current roles and add them to the target list (from `competitor-research`), marked "unverified" until checked on their homepage. People movement reveals companies that other sources miss.

Completion: profile list written; companies from roles extracted when the engagement is company discovery.

## Reference

- recruitin.net (free) builds and runs these queries without an account. Good for paste mode; it also has a paid export feature for large result sets.
- Result count reality check: Google typically returns fewer than 100 profiles per query. If a query returns nothing, loosen in this order: drop the credentials block → drop exclusions → drop the company filter → drop the location phrase.
- Profile URLs: prefer `/in/` profiles (they have public experience); `/pub/` are older and often thinner.

### Operators per engine (x-ray-relevant)

| Operator | Google | Bing | Brave | DDG |
|---|---|---|---|---|
| `site:` | ✅ | ✅ | ✅ | ⚠️ (buggy) |
| `"exact phrase"` | ✅ | ✅ | ✅ | ✅ |
| `-term` (exclude) | ✅ | ✅ | ✅ | ✅ |
| `OR` | ✅ | ✅ (uppercase) | ✅ | ❌ none |
| `intitle:` / `inurl:` | ✅ | ✅ | ✅ | ⚠️ (buggy) |
| `filetype:` | ✅ | ✅ | ✅ | ✅ |

Source: DuckDuckGo's own syntax page (duckduckgo.com/duckduckgo-help-pages/results/syntax) documents the missing `OR` and states some advanced syntax "isn't operating 100% correctly on all queries."

### Verified advanced techniques (2025, tested through Serper/Google)

- **Country-code site:** `site:de.linkedin.com/in/ "sales manager" munich` returns only German profiles (verified 10/10). Combine with location phrases for DACH: `de`, `at`, `ch`.
- **AROUND(n) proximity:** `"sales manager" AROUND(3) "airbus"` returns profiles where both terms sit close together (verified 10/10 relevant, incl. Airbus subsidiaries like STELIA).
- The classic exclusion set (`-intitle:"profiles" -inurl:"dir/"` + degree block) remains the reliable core.

### Dead techniques: do not waste queries on these

2012-era advice that no longer works through Google/Serper (all tested, 0 profile results in 2025):

- `"people you know"`: the profile UI phrase is gone; the string now only matches LinkedIn news/posts pages.
- Wildcard inside phrases: `"current * * engineer"` returns nothing.
- `"viewers"` as isolation term: effectively dead (1 result).
- `site:de.linkedin.com` without the `/in/` path: returns company pages, not profiles.

### Optional experiments (unverified)

- Graduation-year range `2004..2009` (Google range operator): plausible for year-of-graduation filters, untested.
- Bing-vs-Google note from the 2012 source: Bing was "cleaner" for profile isolation (no extra terms needed). Untested on current Bing index; only relevant if a Bing API lane is ever added.

### Engine comparison (x-ray suitability)

| Engine | LinkedIn index depth | Boolean | Automation | Verdict |
|---|---|---|---|---|
| **Google** | deepest | full | only via proxy API (Serper/SerpAPI) | **best results; use via API** |
| **Bing** | good | good | Azure Web Search API (free tier) | decent; use via API, HTML is bot-walled |
| **Brave** | medium | good | free API 2k/mo | good budget pick |
| **Startpage** | = Google | full | none public | Google results w/o Google; paste lane |
| **DuckDuckGo** | = Bing, thin | **no OR** | none | **not for x-ray** |
| **Mojeek** | thin | partial | paid | not for x-ray |
| **Yandex** | good | full | paid, RU account | captcha walls; skip |
