# Melanie's Smoothies: features

A small Streamlit order form backed by Snowflake. A visitor names a smoothie, picks up to five fruits, sees nutrition
data for each fruit from the public Fruityvice API, and submits the order into a Snowflake table. It is a 2024 course
exercise ("Melanie's Smoothies" is the exercise's scenario, not a real shop), kept public as a record of a first data app.

**This is the source-of-truth features list.** Every PR that adds, changes, removes or switches off something a person
can see or use updates this file in the same PR (see "Keeping this file true").

- **Repo:** AlexRosen23/melanies_smoothies (public) · **Folder:** none (no local folder; edited through fresh clones)
- **Status:** Finished course project; small maintenance fixes in November 2026 · **Version:** none · **`main`:** `c9f99d5` (2026-11-15)
- **Last updated:** 2026-11-15, written from the code, README and merged-PR list.

**Status words** (a feature with no status word is live):
| Word | Means |
| --- | --- |
| *Live* | on `main` and works, given a Snowflake connection |
| *Planned* | named in the README's limits, not built |

## At a glance
| | |
| --- | --- |
| Stack | Python, Streamlit 1.54.0 (pinned), Snowflake Snowpark, pandas, requests |
| Code | 1 app file (`streamlit_app.py`, 58 lines) |
| Default URL | `http://localhost:8501` (Streamlit's default) |
| Data sources | Snowflake tables `smoothies.public.fruit_options` and `smoothies.public.orders`; Fruityvice API |
| Tests / CI | none; no workflow files |
| Merged PRs | 5 merged, 1 closed (a Dependabot bump) |

## Contents
1. What it is and who it's for
2. How to run it
3. Architecture at a glance
4. Features
5. Integrations
6. Data and privacy
7. Tests and evidence
8. Known limits and gaps
9. Change log

## 1. What it is and who it's for

- A one-page order form: reads a fruit menu from a warehouse, enriches it from a web API, writes an order back.
- For readers who want to see a small data app end to end. Not a shop: no real orders go through it.

## 2. How to run it

Prerequisites: Python 3.10 or newer, and your own Snowflake account with:
- database `smoothies`, schema `public`;
- table `fruit_options` with columns `FRUIT_NAME` and `SEARCH_ON`;
- table `orders` with at least `ingredients` and `name_on_order`;
- a Streamlit connection named `snowflake` (for example `[connections.snowflake]` in `.streamlit/secrets.toml`, which `.gitignore` excludes).

```bash
git clone https://github.com/AlexRosen23/melanies_smoothies.git
cd melanies_smoothies
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
streamlit run streamlit_app.py
```

- Environment variables: none read by the code. Credentials live in the Streamlit connection settings.
- Without a working connection the page shows the title and name box, then stops with an error at `st.connection("snowflake")`.

## 3. Architecture at a glance

```
browser -> streamlit_app.py -> Snowflake (read fruit_options, insert into orders)
                            -> Fruityvice API (GET /api/fruit/{SEARCH_ON})
```

| Path | What it is |
| --- | --- |
| `streamlit_app.py` | the whole app: UI, Snowflake reads, API calls, order insert |
| `requirements.txt` | dependencies (Streamlit pinned to 1.54.0; the rest unpinned) |
| `.gitignore` | keeps `.streamlit/secrets.toml`, `.env*`, caches and virtual environments out of git |

Streamlit re-runs the script from top to bottom on every interaction.

## 4. Features

### Order form
- **Name box** *(Live)*: a text input "Name On Smoothie:" and an echo line showing the name. `streamlit_app.py`
- **Fruit menu from Snowflake** *(Live)*: reads `FRUIT_NAME` and `SEARCH_ON` from `smoothies.public.fruit_options` into a pandas table. `streamlit_app.py`
- **Choose up to 5 fruits** *(Live)*: a multiselect, `max_selections=5`. `streamlit_app.py`
- **Order summary** *(Live)*: once a fruit is chosen, shows "Your order" with the ingredients and the name. Ingredients are one space-separated string with a trailing space. `streamlit_app.py`

### Nutrition data
- **One table per fruit** *(Live)*: for each chosen fruit, a subheader and a table of the Fruityvice response. `streamlit_app.py`
- **Search term from data** *(Live)*: the `SEARCH_ON` column maps a menu name ("Apples") to the API's term ("Apple"), so the owner of the table can fix a lookup without a code change. The page also prints the search value used. `streamlit_app.py`

### Placing the order
- **Submit Order button** *(Live)*: appears once at least one fruit is chosen. `streamlit_app.py`
- **Parameterized insert** *(Live)*: inserts into `smoothies.public.orders(ingredients, name_on_order)` with bound values (`values (?, ?)`), so a name like `O'Brien` cannot break or change the SQL. Added in PR #3. `streamlit_app.py`
- **Success message** *(Live)*: shows "Your Smoothie is ordered!" after the insert. `streamlit_app.py`

### Not built
- **Input checks** *(Planned, only "if revisited")*: an empty name or empty order is accepted.
- **API error handling** *(Planned)*: no timeout; an error response from Fruityvice is rendered as if it were nutrition data or breaks the page.
- **`width="stretch"`** *(Planned)*: Streamlit 1.54 warns that `use_container_width` is deprecated.
- **Tests** *(Planned)*: none in the repo.

## 5. Integrations

- **Snowflake** (via Snowpark and `st.connection("snowflake")`): reads the menu, writes orders. Needs the user's own account.
- **Fruityvice** (fruityvice.com, public, no key): nutrition data per fruit.
- **Streamlit**: UI and connection layer.

## 6. Data and privacy

- Stores nothing locally. Orders (smoothie name and fruit list) are written to the Snowflake table the user configures.
- Sends the chosen fruit's search term to Fruityvice. The smoothie name is not sent there.
- No credentials in the repo. `.gitignore` excludes `.streamlit/secrets.toml` and `.env*` (added in PR #6 after a 2026-11-10 audit).
- Data is course-exercise data; no real customers or orders.

## 7. Tests and evidence

- No test suite. Counted from source on 2026-11-15, not run.
- Recorded evidence in `README.md` (2026-11-09, Streamlit 1.54.0, Python 3.12): the app was run headless with Streamlit's `AppTest` against a stubbed Snowflake session and the live Fruityvice API. Name `O'Brien` plus 2 fruits produced `values (?, ?)` with bound parameters and the success message; one table per fruit rendered; no exceptions. The stub and script are not committed.
- Not run against a real Snowflake account.
- GitHub Actions is on but there are no workflow files, so nothing runs.

## 8. Known limits and gaps

- No validation of the name or the order.
- No timeout or error handling on the Fruityvice call.
- Ingredients stored as one string, as the course table expects.
- Whether a hosted copy exists is unknown; nothing in the repo records one.
- Only the headless stub run above has been recorded; real-Snowflake behaviour is unverified.
- No license file, so default copyright applies.

## 9. Change log

| Date | PR | Change |
| --- | --- | --- |
| 2026-11-10 | #6 | `.gitignore` for `.streamlit/secrets.toml`, `.env*` and local environments. |
| 2026-11-10 | #5 | Full README write-up. |
| 2026-11-07 | #4 | Professional README. |
| 2026-11-04 | #3 | Fix: bind order values in the INSERT (SQL injection). |
| 2026-11-04 | #2 | Consolidated dependency update (Streamlit 1.31.1 to 1.54.0). |
| 2026-11-04 | #1 (closed) | Dependabot bump to Streamlit 1.54.0, replaced by #2. |
| 2024-06-17 to 18 | none | Original course build in about 26 small web-editor commits: Snowflake session, fruit multiselect, standalone connection, Fruityvice tables, `SEARCH_ON` mapping. |

## Keeping this file true

- **Same PR, same change:** a PR that changes the form, the queries or the dependencies updates this file too.
- **Status moves with reality:** when input checks, error handling or tests are added, move them out of "Not built".
- **Only what is true:** claims about Snowflake behaviour stay "unverified" until run against a real account.
- **Public repo rule:** this file is public. No credentials, no account names, no machine paths.
