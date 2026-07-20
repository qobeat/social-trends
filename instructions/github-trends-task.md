# ChatGPT Scheduled Task — GitHub Trend Snapshots v1

## Recommended task settings

- Title: `GitHub Trend Snapshots`
- Schedule: every 4 hours
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

1. **Web Search** to open public GitHub Trending/Explore pages and exact public
   GitHub repository pages.
2. A public **Browser renderer**, only as a fallback for the exact GitHub URL
   when Web Search returns a page shell such as `Loading` without repository
   entries. The fallback must be unauthenticated: do not log in, accept or set
   cookies, navigate to a substitute page, or change the requested language or
   native window. If the Browser renderer is unavailable, record that limitation
   and continue; its absence is not a global task failure.
3. Connected **GitHub App**:
   - `github_search_repositories` for structured thematic discovery;
   - `github_get_repo` to verify selected repositories;
   - `github_fetch_file` to read the schema, the immediately prior confirmed
     snapshot, and the newly created snapshot;
   - `github_create_file` to create the immutable JSONL snapshot.

Do not use unofficial GitHub Trending APIs as primary evidence. Do not use shell,
local git, browser login, API keys, stored cookies, webhooks, mirrors, cached
substitutes, or hidden endpoints.

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

### Prior-state baseline

1. From the immediately prior task-run context, obtain the exact path of the last
   snapshot whose upload and read-back were confirmed.
2. Fetch that exact path from `main` with `github_fetch_file`. Do not guess a
   path or substitute a non-immediate older snapshot.
3. If no exact prior path is available or read-back fails, record that
   comparison is unavailable and use `trend_state = unknown` for comparisons
   that require it. This does not block collection of the current snapshot.
4. For each source, set
   `source_result.consecutive_unhealthy_runs`:
   - if current status is `blocked` or `stale` and the immediately prior
     status is also `blocked` or `stale`, increment the prior value;
   - if the prior record predates this field, start at `1` rather than
     reconstructing a count from memory;
   - if current status is `blocked` or `stale` after a healthy or unavailable
     baseline, set it to `1`;
   - otherwise set it to `0`.

### Schema and output path

Before collection, fetch from `main`:

`schemas/trend-record.schema.json`

Every non-empty JSONL line must conform to schema version `1.0.0`.

Create one immutable file:

`data/github/YYYY/MM/DD/github-YYYYMMDDTHHMMSSZ.jsonl`

### Stable thematic-query contract

1. Set `search_cutoff_date` to the UTC calendar date seven days before
   `window.end`, formatted `YYYY-MM-DD`.
2. Execute every configured thematic `query_id` in its listed order, requesting
   at most five results from each query. Replace `<search_cutoff_date>`
   literally and append
   `pushed:>=<search_cutoff_date> archived:false fork:false`.
3. Do not put `sort:updated` or any other undocumented sorting token inside a
   search query.
4. Treat the rank returned by one GitHub search call as native only to that
   `query_id`. Store `search_query_id`, the exact rendered `search_query`,
   `search_result_rank`, and `search_cutoff_date` in `native_metrics`.
5. Within a thematic source, retain observations in configured `query_id`
   order and then returned search rank. Deduplicate by canonical repository URL,
   keeping the first occurrence, and cap the retained source sample at 10.
   Record `native_rank = search_result_rank`; record IDs remain sequential and
   must not be presented as native ranks.
6. Never round-robin or otherwise merge several result lists into a synthetic
   native ranking.
7. Rank/trend comparisons require the same source ID, `query_id`, rendered
   cutoff date, and native metric. When the rolling cutoff date changes, use
   `trend_state = unknown` until comparable evidence exists.

### Configured source registry

Attempt every source/query on every run. Preserve duplicate repositories across
different sources because each source represents a distinct signal.

#### 1. GitHub Trending — daily, all languages

- source_id: `github_trending_daily`
- URL: `https://github.com/trending?since=daily`
- Method: Web Search/public page. If the exact response contains only a page
  shell such as `Loading`, make one attempt with the public Browser renderer
  on the same exact URL. If neither method exposes repository entries, record
  the source as `blocked`; do not substitute a language-filtered page, mirror,
  cache, or different native window.
- Capture the first 20 repositories in native order.
- Capture repository full name, URL, rank, description, language, total stars,
  forks, current-period stars, and built-by handles only when exposed.
- Store `native_metrics.native_window = daily`.
- Use `coverage = platform` and `freshness = source_window_broader`.

#### 2. GitHub Trending — weekly, all languages

- source_id: `github_trending_weekly`
- URL: `https://github.com/trending?since=weekly`
- Method: Web Search/public page. Apply the same one-attempt exact-URL public
  Browser fallback and no-substitution rule as the daily source.
- Capture the first 20 repositories in native order.
- Store `native_metrics.native_window = weekly`.
- Use `coverage = platform` and `freshness = source_window_broader`.

#### 3. Agentic AI and deterministic agents

- source_id: `github_search_agents`
- Method: `github_search_repositories`.
- query_id `agents_core`:
  `"AI agent" OR "agentic workflow" OR "deterministic agent"`
- query_id `agents_protocols`:
  `"multi-agent" OR MCP OR "agent orchestration"`
- Retain at most 10 candidates across both fixed queries.
- Set `coverage = query_sample`.

#### 4. AI coding and software lifecycle

- source_id: `github_search_ai_sdlc`
- Method: `github_search_repositories`.
- query_id `ai_sdlc_coding`:
  `"agentic coding" OR "coding agent" OR "AI code review" OR "AI testing"`
- query_id `ai_sdlc_ops`:
  `"software architecture AI" OR "AI ops" OR "SDLC agent"`
- Retain at most 10 candidates across both fixed queries.
- Set `coverage = query_sample`.

#### 5. Local LLM and private AI

- source_id: `github_search_local_ai`
- Method: `github_search_repositories`.
- query_id `local_ai_runtime`:
  `"local LLM" OR Ollama OR "llama.cpp" OR "local inference"`
- query_id `local_ai_private`:
  `"RTX 3090" OR RAG OR "private AI"`
- Retain at most 10 candidates across both fixed queries.
- Set `coverage = query_sample`.

#### 6. AI education and science tools

- source_id: `github_search_education_science`
- Method: `github_search_repositories`.
- query_id `education_core`:
  `"AI education" OR SAT OR "ACT practice" OR "STEM education"`
- query_id `science_agents`:
  `"research agent" OR "arXiv agent" OR "scientific discovery" OR "bioinformatics AI"`
- Retain at most 10 candidates across both fixed queries.
- Set `coverage = query_sample`.

#### 7. Solo-founder business software

- source_id: `github_search_solo_business`
- Method: `github_search_repositories`.
- query_id `solo_saas`:
  `"vertical SaaS" OR "micro SaaS" OR "solo founder"`
- query_id `business_automation`:
  `"business automation" OR "workflow automation"`
- Retain at most 10 candidates across both fixed queries.
- Set `coverage = query_sample`.
- Do not label these broad automation results as HOA/property-management
  coverage unless verified repository metadata independently proves that use
  case.

#### 8. Alex project application tools

- source_id: `github_search_alex_project_apps`
- Method: `github_search_repositories`.
- Execute every query independently with the shared rolling cutoff and common
  qualifiers. Request at most 2 results per query and retain at most 20
  deduplicated candidates in configured query order.
- query_id `offline_trader_analytics`:
  `"broker export" OR "trade journal" OR "portfolio analytics" OR "closed trades" OR "swing trading analytics"`
- query_id `sat_psat_act_prep`:
  `"SAT prep" OR "PSAT prep" OR "ACT prep" OR "test preparation app"`
- query_id `high_school_projects`:
  `"high school project" OR "student research project" OR "science fair" OR "student portfolio app"`
- query_id `niche_web_business_apps`:
  `"small business app" OR "business dashboard" OR "client portal" OR "vertical SaaS"`
- query_id `soundcloud_analytics`:
  `"SoundCloud API" analytics OR "SoundCloud stats" OR "SoundCloud plays" OR "SoundCloud scraper"`
- query_id `youtube_osint`:
  `"YouTube metadata" OR "YouTube channel analysis" OR "YouTube comment scraper" OR "YouTube transcript analysis"`
- query_id `social_network_osint`:
  `"social media OSINT" OR "social graph OSINT" OR "username investigation" OR "social network analysis"`
- query_id `crypto_trading_tools`:
  `"crypto trading analytics" OR "crypto trade journal" OR "crypto market scanner" OR "algorithmic crypto trading"`
- query_id `osint_tools`:
  `"OSINT toolkit" OR "OSINT framework" OR "digital investigation" OR "open source intelligence"`
- query_id `hoa_property_software`:
  `"HOA management" OR "homeowners association" OR "property management software" OR "resident portal"`
- Set `coverage = query_sample`.
- Store `native_metrics.project_area = <query_id>`.
- Do not copy or expose metadata, files, holdings, orders, strategies, or other
  content from Alex's private repositories. The snapshot contains only public
  candidate-repository metadata returned by the configured discovery query.

#### 9. AI runtime, evaluation, and development automation

- source_id: `github_search_ai_dev_tools`
- Method: `github_search_repositories`.
- Execute every query independently with the shared rolling cutoff and common
  qualifiers. Request at most 2 results per query and retain at most 16
  deduplicated candidates in configured query order.
- query_id `ai_agent_runtime`:
  `"AI agent runtime" OR "agent runtime framework" OR "LLM agent runtime" OR "agent execution runtime"`
- query_id `ai_evals`:
  `"AI evals" OR "LLM evals" OR "agent evaluation" OR "evaluation harness"`
- query_id `adlc_agents`:
  `"agent development lifecycle" OR "agentic SDLC" OR "SDLC agent" OR "development lifecycle agent"`
- query_id `subagent_automation`:
  `"subagent automation" OR "sub-agent orchestration" OR "agent delegation" OR "multi-agent development"`
- query_id `codex_dev_automation`:
  `"Codex CLI" OR "OpenAI Codex" OR "Codex automation" OR "Codex skill"`
- query_id `cursor_dev_automation`:
  `"Cursor automation" OR "Cursor rules" OR "Cursor MCP" OR "Cursor plugin"`
- query_id `claude_dev_automation`:
  `"Claude Code plugin" OR "Claude Code skill" OR "Claude Code hook" OR "Claude Code MCP"`
- query_id `mcp_plugins_skills_apps`:
  `"MCP server" OR "agent skill" OR "AI plugin" OR "agent app"`
- Set `coverage = query_sample`.
- Store `native_metrics.project_area = <query_id>`.

For sources 8 and 9:

- a successful individual query returning zero repositories is a valid empty
  query result, not a blocked source;
- use source status `partial` when at least one query succeeds but another
  query fails; use `no_data` only when every query succeeds and all return
  zero retained candidates;
- preserve per-query rank and query identity; never let early query results
  suppress execution of later project areas;
- before a project-specific material notification, verify the candidate and
  require an explicitly relevant public description, topics, or other returned
  metadata. A keyword-only name match is insufficient.

### Repository verification

Use `github_get_repo` for:

- the top 10 daily Trending repositories;
- any repository newly entering the top 10 compared with the prior task run;
- the top 3 retained candidates from each thematic source, selected
  deterministically in configured query order and native search rank;
- every repository evaluated for a cross-group material notification;
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
- `raw_label`: repository description, maximum 1,000 characters. Prefer the
  GitHub App value. If the App omits it, use an exact public repository page via
  Web Search or the guarded public Browser fallback only when the description is
  explicitly visible; otherwise use `null`;
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
  - `search_query_id`;
  - `search_query`;
  - `search_result_rank`;
  - `search_cutoff_date`.

Do not treat `updated_at` as code activity when only metadata changed. Prefer
`pushed_at` or verified commits when available. Missing descriptions or
activity metrics are a coverage limitation, not permission to infer purpose or
acceleration from a repository name. If core identity and URL are verified,
retain the observation with the missing fields omitted or null and add a clear
limitation.

### Trend state

Use a directional state only when directly supported:

- `new`: absent from the directly comparable prior snapshot and now present;
- `rising`: rank improved or a comparable native metric increased;
- `falling`: rank declined;
- `stable`: comparable rank/metric did not materially change;
- `reentered`: seen in an older snapshot, absent in the immediately prior one,
  and now present again;
- `unknown`: no directly comparable prior evidence.

Comparisons must use the same source ID, `search_query_id`, exact rendered
query including cutoff date, native window, and metric. Do not infer
acceleration from total stars, repository names, or result-set churn alone.

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
   `validation_status = not_available`. This is a recorded capability
   limitation, not a validation failure, when all structural checks pass.
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

Compare source health with the immediately prior snapshot:

- a source-health event is material when `ok` or `partial` changes to
  `blocked` or `stale`;
- recovery from `blocked` or `stale` to `ok` or `partial` is material;
- when no prior baseline exists, notify once if the current source is blocked
  or stale;
- do not notify merely because `blocked` or `stale` remains unchanged;
- for an uninterrupted failure, send at most one escalation at six consecutive
  failed runs and one additional reminder at each multiple of 24 runs.

Notify Alex in Russian only when at least one is true:

- a repository newly enters the daily top 10;
- a repository moves at least five daily ranks;
- a new relevant repository is present in at least two independent thematic
  source groups in both the current and immediately prior comparable run, and
  verified description, topics, or activity metadata confirms relevance;
- a repository relevant to ADOS, agentic coding, local LLMs, AI runtimes or
  evals, ADLC/SDLC agents, subagent automation, Codex/Cursor/Claude development
  automation, MCP/plugins/skills/apps, offline trader analytics, SAT/PSAT/ACT
  preparation, high-school projects, niche web business apps, SoundCloud or
  YouTube analytics, social-network OSINT, crypto trading, general OSINT,
  HOA/property automation, or solo-founder software shows confirmed material
  acceleration based on directly comparable evidence;
- a configured source has a material health transition or reaches one of the
  failure-reminder thresholds above;
- actual JSONL validation fails, or GitHub upload/read-back verification fails.

The following are not material by themselves:

- no repository appears in two thematic groups;
- thematic result-set or rank churn without comparable query evidence;
- `validation_status = not_available` when structural validation passes;
- an unchanged source limitation already reported in the prior run;
- an empty but successfully executed project-area query;
- a broad or ambiguous query result without verified relevance to its recorded
  `project_area`.

The notification must include:

- New York timestamp and run ID;
- repository, observed change, and why it matters;
- for project discovery, the exact `project_area` and the Alex project/use case
  it may support;
- confirmed facts separated from inference;
- direct GitHub URLs;
- source/query coverage limitations relevant to the material event;
- snapshot path and upload verification.

If no material condition exists and storage succeeds, emit no user-facing
notification. Keep unchanged limitations in the raw JSONL rather than repeating
them hourly. Do not include sentiment in version 1.
