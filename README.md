# Melanie's Smoothies

A small Streamlit app where you pick up to five fruits, see their nutrition data and submit a smoothie order into a Snowflake table.

**Status:** Finished course project (2024), with small maintenance updates in 2026 · **Stack:** Python, Streamlit, Snowflake (Snowpark), pandas · Written by hand in 2024 as a Snowflake and Streamlit course project; 2026 updates made with [Claude Code](https://claude.com/claude-code)

## What it does

You type a name for the smoothie, choose up to five fruits from a list, and the app shows nutrition information for each fruit. Pressing Submit Order writes the order to a table in Snowflake.

The app was written by hand in June 2024 while working through a Snowflake and Streamlit course. In November 2026 Claude Code was used for two small changes: a consolidated dependency update, and a security fix so that order values are bound as parameters instead of being pasted into the SQL statement.

## Key features

- Fruit list loaded from a Snowflake table through a Snowpark session.
- Multi-select limited to five ingredients.
- Nutrition lookup for each chosen fruit from the public Fruityvice API.
- Order submission into a Snowflake `orders` table.
- Parameterized INSERT, so names such as `O'Brien` cannot break the statement.

## How it works

```
streamlit_app.py    the whole app: UI, Snowflake reads, API calls, order insert
requirements.txt    dependencies
```

The app reads fruit names and API search terms from a `fruit_options` table, calls the Fruityvice API for each selection, and inserts the finished order into an `orders` table. It connects through Streamlit's `snowflake` connection, which is configured outside the repository.

## Built with

Python, Streamlit, Snowflake Snowpark, pandas and requests (see `requirements.txt`). The original app was hand-written; Claude Code helped with the 2026 dependency and SQL-binding changes.

## Run it locally

Prerequisites: Python 3, and a Snowflake account with the `fruit_options` and `orders` tables set up and a `snowflake` connection configured for Streamlit (credentials are never stored in this repository).

```bash
pip install -r requirements.txt
streamlit run streamlit_app.py
```

## Project stats

| | |
|---|---|
| Timeline | 2024-06-17 → 2026-11-04 (first to last commit) |
| Commits | 30 |
| Active Claude Code time | 1.3 hours across 2 sessions (2026-11-03 → 2026-11-04) |
| Claude tokens (new work) | 210 thousand: 166 input, 164 thousand written to cache, 45 thousand output |
| Claude tokens (cache reads) | 12.8 million, counted separately (see below) |
| Models (share of responses) | Claude Opus 5.5 100% |

Claude Code transcripts on the current build machine start on 2026-10-20; earlier sessions were not carried over, so time and token figures cover 2026-10-20 onward only. The commit timeline covers the whole project.

<details>
<summary>How these numbers were measured</summary>

Measured from Claude Code session transcripts (the `.jsonl` files Claude Code writes for every session and subagent) with a small read-only script. Each Claude response's `usage` record is counted once. A response counts toward this repo when its working directory is inside the repo (or a git worktree of it); for sessions started from a general folder, when its tool calls point into the repo, or else the project that session was last working on.

- **New-work tokens** = input tokens + tokens written to the prompt cache + output tokens.
- **Cache reads** are the conversation context that Claude re-reads from Anthropic's prompt cache on every turn. They are far larger than new work and are billed at about a tenth of the input price, so they are listed separately rather than added to the total.
- **Active time** joins transcript events that are at most 15 minutes apart into working intervals, then merges overlapping intervals so parallel sessions and subagents are not double counted. Idle gaps longer than 15 minutes are not counted.

All figures are approximate.

</details>

## Notes

This is a learning project. The Snowflake connection details are not part of the repository, so the app will not run without your own Snowflake setup.
