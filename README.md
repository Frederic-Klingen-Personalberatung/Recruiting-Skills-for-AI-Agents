# Recruiting Skills for AI Agents

Practical skills that teach AI agents how experienced headhunters actually
research a market and find people, not just how to send more messages.

Written by Frederic Klingen, executive search consultant since 2007 with a focus
on automotive, engineering, electronics and defense.
[LinkedIn](https://www.linkedin.com/in/headhunterautomotive)

## Why

AI agents can now search the web, read company pages and look for candidates.
They know a lot about research in theory. What they don't do reliably is apply
it the way an experienced researcher would: they stop after the first good
results, quietly narrow the scope, skip the step of verifying, and miss the
obvious big names. These skills put that discipline into the agent's
instructions, so it does the groundwork properly and the human can focus on
the conversations.

This is a small collection of examples from my daily work, not a complete
toolkit. The skills are tested in real searches and updated whenever I learn
something new.

## Skills

| Skill | What it does |
|---|---|
| [`competitor-research`](competitor-research/SKILL.md) | Goes from a client to its competitors, technology area and market. Uses product pages, trade fairs, associations, job ads, web search and LinkedIn, verifies every company and checks the list for completeness. Output: a verified target list of companies. |
| [`xray`](xray/SKILL.md) | Finds public LinkedIn profiles via site-restricted web search (x-ray), without logging in to LinkedIn. Includes a query builder, tested operators and techniques that no longer work. |

The method behind them is useful beyond recruiting: competitor and market
research works the same way, recruiting just adds the people as a source.

## How to use

Each skill is a folder with a `SKILL.md` file.

- **Claude:** add the skill folder as a skill in Claude, or place it in your
  agent's skills directory (e.g. for Claude Code: `.claude/skills/`).
- **Other agents:** give the `SKILL.md` file to the agent as instructions or
  system prompt. The skills are plain text and work with any capable model.

The `xray` skill works best with a search API (e.g. Serper). Set your own key
as an environment variable. Never put API keys into the files.

## License

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/): free to use and
adapt, including commercially, as long as you credit the author.

## Contact

Questions, feedback or a project in mind? Reach me on
[LinkedIn](https://www.linkedin.com/in/headhunterautomotive).
