# arXiv Talent Radar n8n + Claude + GitHub + Airtable

![arXiv Talent Radar Workflow](arXiv%20n8n%20Talent%20Radar.png)

## What this project does

This project builds an automated technical-sourcing system that runs every morning. It reads the daily [arXiv cs.LG](http://arxiv.org/rss/cs.LG) research feed, uses Claude to evaluate whether each paper's authors match your hiring target, looks up the first author on GitHub, and saves qualified candidates into Airtable with a fit score and a personalized outreach line.

After setup, the process is hands-off. Each run produces a fresh, relevance-ranked candidate shortlist ready for recruiter review and outreach.

## Why arXiv

Great engineers publish. New ML papers appear on arXiv every day, but manually reading papers, judging author fit, and tracking down profiles is slow and inconsistent.

This workflow turns that into a repeatable system with consistent screening criteria, structured AI evaluation, and clean candidate tracking.

## Workflow logic explained

This workflow runs on a recurring schedule and executes the same sourcing pipeline every cycle.

It starts from new research papers in a selected arXiv feed, then extracts authors and processes candidates one by one.

Each candidate is enriched with publication context and GitHub profile signals, then sent to Claude for structured scoring across fit, strengths, gaps, and recommended next action.

Only candidates that pass your quality threshold are written to Airtable with sourcing metadata and outreach-ready notes.

The result is a repeatable system that turns raw research activity into a prioritized, recruiter-usable shortlist.

## Who this is for (and not for)

### This is for you if
- You recruit machine learning and research engineering talent and want to discover people based on real technical output.
- You want a repeatable pipeline that identifies authors from relevant arXiv papers and prioritizes who to contact first.
- You want AI-assisted scoring and outreach hooks grounded in publication recency, topic relevance, and technical depth.

### This is not for you if
- You only recruit non-technical roles where publication signals are low value.
- You cannot use external AI APIs due to internal compliance or policy constraints.

## The search engines this workflow runs

This workflow uses arXiv feed + AI evaluation as its sourcing engine, then enriches results with GitHub.

### Search 1 — arXiv Feed Intake
Pulls fresh papers from a selected arXiv category feed (default: `cs.LG`).

Why it matters: Captures active and recent technical contributors.

### Search 2 — Author Extraction
Extracts paper authors and normalizes candidate records.

Why it matters: Converts paper-level data into person-level sourcing entries.

### Search 3 — AI Relevance Scoring
Claude evaluates role fit based on paper content and author signals.

Why it matters: Applies consistent screening criteria at scale.

### Search 4 — GitHub Enrichment
Finds the first author on GitHub and adds profile context.

Why it matters: Adds practical engineering signals and outreach context.

Note: See how customize the searches for your role below. 

## Accounts you need to create

1. **n8n** (n8n.io), the automation engine that connects everything.
2. **Anthropic** (console.anthropic.com), Claude AI scoring.
3. **GitHub** (github.com), author profile lookup.
4. **Airtable** (airtable.com), your candidate database.

## How to get each API key

- **Anthropic**: console.anthropic.com → API Keys → Create Key.
- **GitHub**: Settings → Developer settings → Personal access tokens → fine-grained token with read access for profile lookup.
- **Airtable**: airtable.com/create/tokens → token with `data.records:write` and `schema.bases:read` for your base.
- **n8n**: no API key required for n8n itself; create credentials in n8n for Anthropic, GitHub, and Airtable.

> Never paste API keys into workflow JSON or `PROMPT.md`. Store secrets only in n8n credentials.

## Quickstart — two ways to build it

### Option A, Import the ready-made workflow
1. In n8n, click **Add workflow → Import from File** (or **Import from URL**).
2. Import `workflow/arxiv-talent-radar.json` from this repo.
3. Connect your credentials and set your Airtable destination (see Setup details below).
4. Run a manual test, verify rows in Airtable, then activate.

### Option B, Rebuild it from a single prompt
1. Open n8n and create a new workflow.
2. Open the **n8n AI Assistant**.
3. Paste the full prompt from [`PROMPT.md`](PROMPT.md).
4. Connect credentials, test, and activate.

`PROMPT.md` describes every node so Claude / the n8n AI Assistant can reconstruct the workflow from scratch — handy if you want to understand or customize each step.

## How the workflow operates

1. A scheduled RSS trigger reads new papers from the arXiv feed.
2. A test-limit node caps run size while testing.
3. Claude evaluates each paper against your target role and returns structured JSON.
4. A parse step extracts the JSON; an IF node keeps only relevant papers.
5. GitHub is searched for the first author's profile.
6. A code node merges Claude output, GitHub match, and paper metadata.
7. Qualified candidates are saved to Airtable.

## Airtable fields required by this workflow

Fields the workflow **writes**:

- full_name
- github_url
- source
- date_sourced
- fit_score
- key_strengths
- key_gaps
- outreach_hook
- profile_summary
- paper_title
- paper_url
- institution
- author_position
- career_stage
- arxiv_category
- relevance_score
- co_authors

Optional **manual tracking** columns (not filled by the workflow — add them if you want to track outreach in Airtable):

- current_company
- contacted
- replied
  
## Setup details

### 1) Import workflow
In n8n, import `workflow/arxiv-talent-radar.json`.

### 2) Connect credentials
This template ships with **no credentials attached**. On import, connect your own:
- **Anthropic** (`anthropicApi`), used by **Claude Relevance Check**.
- **GitHub** (`githubApi`), used by **Find Author on GitHub**.
- **Airtable** (`airtableTokenApi`), used by **Save to Airtable**.

### 3) Point at your Airtable base
Open the **Save to Airtable** node and set:
- `YOUR_AIRTABLE_BASE_ID`
- `YOUR_AIRTABLE_TABLE_ID`

Or choose from dropdowns once Airtable credentials are connected.

### 4) Configure the role you're hiring for
Open **Claude Relevance Check** and edit the `ROLE I AM HIRING FOR` line in the prompt.

### 5) Run test
Execute one manual run, confirm records are written to Airtable, and review summary quality.

### 6) Activate automation
Remove or raise the **Limit to 2 (Test)** cap, then activate the workflow.

### 7) Schedule
Default run is daily at **06:00** in your n8n instance timezone. Change this in the trigger node.

## Customizing

- **Different research area**: swap the feed URL (for example `cs.CL`, `cs.CV`, `stat.ML`).
- **Model/cost**: change the `model` field in **Claude Relevance Check**.
- **Scoring strictness**: tune prompt criteria and relevance threshold.

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

## Security notes

This repo is a public workflow template. It should contain logic only, never live secrets or personal data.

What is safe to publish: workflow structure, node logic, public URLs, field mappings, and credential type references.

What must not be committed: API keys, tokens, n8n credential IDs, webhook secrets, `.env` files, private keys, or candidate personal contact data.

Use placeholders like `YOUR_AIRTABLE_BASE_ID` and `YOUR_AIRTABLE_TABLE_ID`.

Gitleaks runs via `.github/workflows/gitleaks.yml`. Run locally before push:

`gitleaks detect --config .gitleaks.toml --source . --no-git -v`


## License

MIT.
