# ChatGPT Scheduled Task — GitHub Trend Snapshots v1

## Recommended task settings

- Title: `GitHub Trend Snapshots`
- Schedule: every hour
- Mode: monitoring/condition watch
- Output language: Russian
- Repository: `qobeat/social-trends`
- Branch: `main`

Paste the instruction below into a separate ChatGPT Scheduled Task.

---

## Task instruction

Run one GitHub-trend collection iteration. This is a raw repository-trend
collection task, not a repository review, sentiment task, or general AI-news
digest.

### Goal

Capture the current GitHub Trending lists and newly accelerating repositories in
Alex's priority domains, save the complete raw snapshot as immutable JSONL in
`qobeat/social-trends`, and notify Alex in Russian only about material changes
or collection/storage failures.

### Required tools

Use only:

1. **Web Search** to open public GitHub Trending/Explore pages.
2. Connected **GitHub App**:
   - `github_search_repositories` for structured thematic discovery;
   - `github_get_repo` to verify selected repositories;
   - `github_fetch_file` to read the schema and verify the snapshot;
   - `github_create_file` to create the immutable JSONL snapshot.

Do not use unofficial GitHub Trending APIs as primary evidence. Do not use shell,
local git, browser login, API keys, cookies, webhooks, or hidden endpoints.

Never use `github_update_file` or `github_delete_file` under `data/`.

If Web Search or the GitHub App is unavailable, report the blocker. Do not claim
that collection or upload succeeded.

### Time window

1. Get current UTC and America/New_York time.
2. Set `window.end` to current UTC time.
3. Set `window.start` to exactly three hours earlier.
4. Set `lookback_hours = 3`.
5. Set:
   - `run_id = github-YYYYMMDDTHHMMSSZ`;
   - `task_id = github-trends-v1`;
   - `collected_at = window.end`.

GitHub's native daily/weekly trends are broader than three hours. Preserve their
native window and use `freshness = source_window_broader`. The hourly snapshots
allow later calculation of three-hour rank changes.

### Schema and output path

Before collection, fetch from `main`:

`schemas/trend-record.schema.json`

Every non-empty JSONL line must conform to schema version `1.0.0`.

Create one immutable file:

`data/github/YYYY/MM/DD/github-YYYYMMDDTHHMMSSZ.jsonl`

### Configured source registry

Attempt every source/query on every run. Preserve duplicate repositories across
different sources because each source represents a distinct signal.

#### 1. GitHub Trending — daily, all languages

- source_id: `github_trending_daily`
- URL: `https://github.com/trending?since=daily`
- Method: Web Search/public page.
- Capture the first 20 repositories in native order.
- Capture repository full name, URL, rank, description, language, total stars,
  forks, current-period stars, and built-by handles only when exposed.
- Store `native_metrics.native_window = daily`.
- Use `coverage = platform` and `freshness = source_window_broader`.

#### 2. GitHub Trending — weekly, all languages

- source_id: `github_trending_weekly`
- URL: `https://github.com/trending?since=weekly`
- Method: Web Search/public page.
- Capture the first 20 repositories in native order.
- Store `native_metrics.native_window = weekly`.
- Use `coverage = platform` and `freshness = source_window_broader`.

#### 3. Agentic AI and deterministic agents

- source_id: `github_search_agents`
- Method: `github_search_repositories`.
- Search query should combine current repository-search qualifiers with:
  `AI agent`, `agentic workflow`, `deterministic agent`, `multi-agent`,
  `MCP`, and `agent orchestration`.
- Prefer repositories created or materially pushed in the last seven days.
- Retrieve at most 10 candidates.
- Set `coverage = query_sample`.

#### 4. AI coding and software lifecycle

- source_id: `github_search_ai_sdlc`
- Method: `github_search_repositories`.
- Search for `agentic coding`, `coding agent`, `AI code review`,
  `AI testing`, `software architecture AI`, `AI ops`, and `SDLC agent`.
- Prefer repositories created or pushed in the last seven days.
- Retrieve at most 10 candidates.
- Set `coverage = query_sample`.

#### 5. Local LLM and private AI

- source_id: `github_search_local_ai`
- Method: `github_search_repositories`.
- Search for `local LLM`, `Ollama`, `llama.cpp`, `local inference`,
  `RTX 3090`, `RAG`, and `private AI`.
- Prefer repositories created or pushed in the last seven days.
- Retrieve at most 10 candidates.
- Set `coverage = query_sample`.

#### 6. AI education and science tools

- source_id: `github_search_education_science`
- Method: `github_search_repositories`.
- Search for `AI education`, `SAT`, `ACT practice`, `STEM education`,
  `research agent`, `arXiv agent`, `scientific discovery`, and
  `bioinformatics AI`.
- Prefer repositories created or pushed in the last seven days.
- Retrieve at most 10 candidates.
- Set `coverage = query_sample`.

#### 7. Solo-founder business software

- source_id: `github_search_solo_business`
- Method: `github_search_repositories`.
- Search for `vertical SaaS`, `micro SaaS`, `solo founder`,
  `business automation`, `HOA management`, `property management AI`, and
  `workflow automation`.
- Prefer repositories created or pushed in the last seven days.
- Retrieve at most 10 candidates.
- Set `coverage = query_sample`.

### Repository verification

Use `github_get_repo` for:

- the top 10 daily Trending repositories;
- any repository newly entering the top 10 compared with the prior task run;
- the top 3 candidates from each thematic search;
- any repository selected for the user notification.

If verification fails, retain the public-page observation only when its URL and
label are clear, add a limitation, and record the verification failure in the
source-status message. Do not invent GitHub metadata.

### Observation mapping

For each repository:

- `topic`: repository full name, e.g. `owner/repo`;
- `canonical_topic`: same normalized lowercase full name;
- `item_url`: canonical repository URL;
- `native_id`: GitHub repository numeric ID when returned, otherwise `null`;
- `native_rank`: native Trending rank or search-result rank;
- `raw_label`: repository description, maximum 1,000 characters;
- `native_metrics`: only native values actually returned, such as:
  - `language`;
  - `stars_total`;
  - `stars_current_period`;
  - `forks`;
  - `open_issues`;
  - `created_at`;
  - `updated_at`;
  - `pushed_at`;
  - `native_window`;
  - `search_query`.

Do not treat `updated_at` as code activity when only metadata changed. Prefer
`pushed_at` or verified commits when available.

### Trend state

Use a directional state only when directly supported:

- `new`: absent from the directly comparable prior snapshot and now present;
- `rising`: rank improved or a comparable native metric increased;
- `falling`: rank declined;
- `stable`: comparable rank/metric did not materially change;
- `reentered`: seen in an older snapshot, absent in the immediately prior one,
  and now present again;
- `unknown`: no directly comparable prior evidence.

Comparisons must use the same source ID, query, native window, and metric.
Do not infer acceleration from total stars alone.

### JSONL construction

Create records in this order:

1. one `run_start`;
2. all `trend_observation` records in configured source order and native rank;
3. exactly one `source_status` per configured source;
4. one `run_end`.

Use record IDs:

- `<run_id>/run_start/000`
- `<run_id>/<source_id>/trend/NNN`
- `<run_id>/<source_id>/status/000`
- `<run_id>/run_end/000`

Within one source and run, deduplicate by canonical repository URL. Do not
deduplicate across different sources or runs.

Do not store README text, source code, issues, comments, user emails, credentials,
or private-repository data. This task collects public trend metadata only.

### Validation

1. Produce compact JSONL with one object per line and a final newline.
2. Confirm that every line parses independently.
3. Check required fields, enums, UTC timestamp formats, output-path pattern,
   record ordering, unique IDs, and one status per source.
4. If a JSON Schema validator is actually available, validate every line and use
   `validation_status = passed` only after success.
5. Otherwise perform structural checks and use
   `validation_status = not_available`.
6. Do not upload malformed JSONL.

### GitHub upload

Call `github_create_file` with:

- repository: `qobeat/social-trends`
- branch: `main`
- path: immutable output path
- message: `data(github): add <run_id> snapshot`
- content: complete JSONL text

If that path exists, never overwrite it. Report the collision and stop.

After creation, call `github_fetch_file` on the new path from `main`. Verify
first record ID, last record ID, and non-empty line count. Only then report that
upload succeeded.

### Material-update gate

Always save the raw snapshot when collection and GitHub write tools work.

Notify Alex in Russian only when at least one is true:

- a repository newly enters the daily top 10;
- a repository moves at least five daily ranks;
- a new relevant repository appears in at least two independent source/query
  groups;
- a repository relevant to ADOS, agentic coding, local LLMs, kids education,
  frontier science, HOA/property automation, or solo-founder software shows a
  confirmed material acceleration;
- a configured source becomes blocked or stale;
- JSONL validation, GitHub upload, or read-back verification fails.

The notification must include:

- New York timestamp and run ID;
- repository, observed change, and why it matters;
- confirmed facts separated from inference;
- direct GitHub URLs;
- source/query coverage limitations;
- snapshot path and upload verification.

Do not include sentiment in version 1.
