# arXiv Talent Radar n8n + Claude + GitHub + Airtable

![arXiv Talent Radar Workflow](arXiv%20n8n%20Talent%20Radar.png)

## What this project does, read this first

This project builds an automated technical-sourcing system that runs every morning. It reads the daily [arXiv cs.LG](http://arxiv.org/rss/cs.LG) research feed, uses **Claude** to decide whether each paper's authors are a fit for a role you're hiring for, looks up the first author on **GitHub**, and saves qualified candidates into **Airtable**, each with a fit score and a personalized outreach line.

After setup, the process is hands-off. Each morning, Airtable is pre-filled with fresh, relevance-ranked candidates ready for outreach.

## Why this exists

Great engineers publish. New ML papers appear on arXiv every day, but manually reading them, judging author fit, and tracking down profiles is slow and inconsistent. This workflow turns that into a repeatable system: consistent screening criteria, structured AI evaluation, and clean candidate tracking.

## 4 accounts you need to create

1. **n8n**, n8n.io, the automation engine that connects everything
2. **Anthropic**, console.anthropic.com, Claude AI scoring
3. **GitHub**, github.com, author profile lookup
4. **Airtable**, airtable.com, your candidate database

## 1-minute quickstart

1. Open n8n and create a new workflow.
2. Open the n8n AI Assistant.
3. Paste the full prompt from `PROMPT.md`.
4. Connect Anthropic, GitHub, and Airtable credentials.
5. Run a manual test and verify rows in Airtable.
6. Activate the workflow.

> Prefer importing over rebuilding? Use `workflow/arxiv-talent-radar.json` (see Setup details).

## How the workflow operates

1. A daily RSS trigger reads new papers from the arXiv cs.LG feed.
2. A test-limit node caps the run size while you're testing.
3. Claude evaluates each paper against your target role and returns structured JSON.
4. A parse step extracts the JSON; an IF node keeps only relevant papers.
5. GitHub is searched for the first author's profile.
6. A code node merges Claude output, the GitHub match, and paper metadata.
7. Qualified candidates are saved to Airtable.

## Repository structure
```text
workflow/
  arxiv-talent-radar.json
.github/
  workflows/
    gitleaks.yml
README.md
PROMPT.md
.gitleaks.toml
.gitignore
LICENSE
```

## Airtable fields required by this workflow

- full_name
- github_url
- source
- date_sourced
- fit_score
- key_strengths
- key_gaps
- outreach_hook
- profile_summary

## Setup details

### 1) Import workflow

In n8n, import `workflow/arxiv-talent-radar.json`.

### 2) Connect credentials

Add credentials for Anthropic, GitHub, and Airtable in n8n, then attach them:

- **Anthropic** (`anthropicApi`), used by **Claude Relevance Check**
- **GitHub** (`githubApi`), used by **Find Author on GitHub**
- **Airtable** (`airtableTokenApi`), used by **Save to Airtable**

### 3) Point at your Airtable base

Open the **Save to Airtable** node and set your own destination:

- `YOUR_AIRTABLE_BASE_ID` → your Airtable base
- `YOUR_AIRTABLE_TABLE_ID` → your Airtable table

Or select them from the dropdowns once your Airtable credential is connected. Make sure your table includes the fields listed above.

### 4) Configure the role you're hiring for

Open **Claude Relevance Check** and edit the `ROLE I AM HIRING FOR` line in the prompt. The default is:

> Senior ML Engineer. Must have: PyTorch or JAX, LLM training or fine-tuning, distributed systems knowledge

### 5) Run test

Execute one manual run, confirm records are written to Airtable, review summary quality.

### 6) Activate automation

Delete the **Limit to 2 (Test)** node (or raise `maxItems`) so full runs process, then turn on the workflow. It runs daily at **06:00** in your n8n instance timezone — change this in the **arXiv cs.LG Feed** node.

## Customizing

- **Different research area** — swap the feed URL in **arXiv cs.LG Feed** (e.g. `cs.CL`, `cs.CV`, `stat.ML`). See the [arXiv RSS list](https://arxiv.org/help/rss).
- **Model / cost** — change the `model` field in **Claude Relevance Check**.
- **Scoring strictness** — tune the system prompt and the relevance threshold.

## Security notes

- This repo contains **no API keys or tokens**. Credentials live in n8n's encrypted credential store and are never exported to the workflow JSON.
- Airtable base and table IDs are placeholders — no private destination is exposed.
- Never commit a `.env` file or paste secrets into the workflow JSON. Keep keys in n8n credentials or a secrets manager only.

## Operating notes

- Tune the role prompt and filters weekly to improve candidate quality.
- Add deduplication logic to avoid repeated authors.
- Use status fields in Airtable to track outreach progress.

## License

MIT.
