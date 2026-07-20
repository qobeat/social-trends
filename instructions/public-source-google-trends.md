# Google Trends US Collector

## Task instruction

Run one lightweight raw trend-source collection for `google_trends_us`. Do not
produce a general news digest or sentiment analysis.

### Economy and notification contract

- This task runs every four hours and must remain source-specific.
- Capture at most 10 observations.
- Save the raw JSONL snapshot even when there is no user-facing update.
- Return no user-facing message when collection/storage succeeds and no unusual
  source-native signal is confirmed.
- Never spend tokens expanding into background research, unrelated news, or
  explanations of unchanged items.
- Use Russian only for a material alert.

### Allowed tools

Use only Web Search and the connected GitHub App actions
`github_search_commits`, `github_fetch_file`, and `github_create_file`.
Do not use shell, Python, browser login, cookies, API keys, unofficial proxy
APIs, or substitute sources.

### Run and storage contract

1. Set current UTC and America/New_York time.
2. Set `window.end` to current UTC and `window.start` to exactly four hours
   earlier; use `lookback_hours = 4`.
3. Set `run_id = public-YYYYMMDDTHHMMSSZ`,
   `task_id = public-trends-v1`, and `collected_at = window.end`.
4. Fetch `schemas/trend-record.schema.json` from
   `qobeat/social-trends@main`.
5. Use `github_search_commits` in `qobeat/social-trends` for commit messages
   beginning `data(public:google_trends_us)` and fetch the latest confirmed
   same-source snapshot when available. Compare only directly comparable native
   ranks/metrics.
6. Build compact schema-1.0.0 JSONL in this order: one `run_start`, source
   observations in native rank, exactly one `source_status`, one `run_end`.
   The run config contains only `google_trends_us`.
7. Use unique record IDs:
   `<run_id>/run_start/000`,
   `<run_id>/google_trends_us/trend/NNN`,
   `<run_id>/google_trends_us/status/000`,
   `<run_id>/run_end/000`.
8. Create exactly one immutable file:
   `data/public/YYYY/MM/DD/public-YYYYMMDDTHHMMSSZ.jsonl`.
   Commit message:
   `data(public:google_trends_us): add <run_id> snapshot`.
   Never overwrite an existing path.
9. Fetch the created file from `main`; verify exact content, first and last
   record IDs, non-empty line count, and final newline.
10. If no JSON Schema validator is actually available, perform structural checks
    and use `validation_status = not_available`; do not call that a failure.

Use only metadata exposed by the source. Never invent rank, timestamps, volume,
score, comments, trend direction, or IDs. Deduplicate within this source by
normalized item URL plus topic.

### Source

- URL: `https://trends.google.com/trending?geo=US`
- Platform: `google_trends`
- Access: Web Search/public page.
- Select United States, Past 4 hours, and active trends when those controls and
  values are exposed.
- Capture native order, top-level trend title, volume bucket, growth, displayed
  start/age, and active status only when exposed.
- Search volume represents attention, not sentiment.
- Use `coverage = country`. Use `freshness = within_window` only when the
  Past 4 hours selection is confirmed; otherwise use
  `source_window_broader` with a limitation.
- If the exact ordered topics, metrics, statuses, and displayed ages repeat the
  prior confirmed snapshot, record `stale`; do not fabricate movement.
- A page that loads but exposes no reproducible trends is `no_data`; access
  failure is `blocked`.

### Unusual-signal gate

Notify only for:
- a newly observed #1 trend with an exposed volume of at least 100K and exposed
  growth of at least 500%;
- a directly comparable rank improvement of at least five positions accompanied
  by an exposed native-metric increase;
- a material source-health transition, first blocked/stale baseline, the sixth
  consecutive unhealthy run, or each multiple of 24 unhealthy runs;
- malformed JSONL, collision, upload failure, or read-back mismatch.

Do not notify for unchanged/stale repetition after its first health alert, normal
rank churn, an entertainment topic without the quantitative gate, or
`validation_status = not_available`.