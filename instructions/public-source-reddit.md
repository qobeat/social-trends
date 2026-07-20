# Reddit Popular US Collector

## Task instruction

Run one lightweight raw trend-source collection for `reddit_popular_us`. Do not
produce a general news digest or sentiment analysis.

### Economy and notification contract

- This task runs every four hours and must remain source-specific.
- Capture at most 10 observations.
- Save the raw JSONL snapshot even when there is no user-facing update.
- Return no user-facing message when collection/storage succeeds and no unusual
  source-native signal is confirmed.
- Never spend tokens expanding into background research, unrelated news, or
  explanations of unchanged items.
- Use Russian only for a material alert or failure diagnostic.

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
   beginning `data(public:reddit_popular_us)` and fetch the latest confirmed
   same-source snapshot when available. Compare only directly comparable native
   ranks/metrics.
6. Build compact schema-1.0.0 JSONL in this order: one `run_start`, source
   observations in native rank, exactly one `source_status`, one `run_end`.
   The run config contains only `reddit_popular_us`.
7. Use unique record IDs:
   `<run_id>/run_start/000`,
   `<run_id>/reddit_popular_us/trend/NNN`,
   `<run_id>/reddit_popular_us/status/000`,
   `<run_id>/run_end/000`.
8. Create exactly one immutable file:
   `data/public/YYYY/MM/DD/public-YYYYMMDDTHHMMSSZ.jsonl`.
   Commit message:
   `data(public:reddit_popular_us): add <run_id> snapshot`.
   Never overwrite an existing path.
9. Fetch the created file from `main`; verify exact content, first and last
   record IDs, non-empty line count, and final newline.
10. If no JSON Schema validator is actually available, perform structural checks
    and use `validation_status = not_available`; do not call that a failure.

Use only metadata exposed by the source. Never invent rank, timestamps, volume,
score, comments, trend direction, or IDs. Deduplicate within this source by
normalized item URL plus topic.

### Diagnostic and failure contract

Track the current stage and the last successfully completed stage using exactly
these stage names:

`bootstrap`, `instruction_fetch`, `schema_fetch`,
`prior_snapshot_lookup`, `source_fetch_popular`,
`source_fetch_rising`, `snapshot_build`, `validation`,
`snapshot_write`, `readback`, `completion`.

Handle failures deterministically:

1. Fetch the Popular and Rising pages independently. Failure of one page must
   not prevent trying the other page.
2. A source page that is blocked, inaccessible, empty, or not reproducibly
   parseable is source data, not an unhandled task exception.
3. If at least one page works, write a normal snapshot with
   `source_result.status = partial`.
4. If neither page works, still write a zero-observation normal snapshot when a
   schema-valid snapshot can be built. Use `blocked` for access denial and
   `error` for a tool/runtime failure. Put a compact sanitized diagnostic in
   `source_result.message`:
   `stage=<stage>; code=<error_code>; retryable=<true|false>; detail=<summary>`.
   Keep it within the schema's 1000-character limit.
5. Missing scores, comment counts, ages, ranks, or HTTP status values are not
   task failures. Omit unavailable optional values and never invent them.
6. A hard failure is one that prevents a valid normal snapshot or its verified
   storage: instruction/schema failure, snapshot-build failure, malformed
   JSONL, validation failure, collision, GitHub write failure, or read-back
   mismatch.
7. On a hard failure, if the GitHub App is still usable, fetch
   `schemas/collector-diagnostic.schema.json` and create exactly one
   immutable diagnostic JSON file:
   `data/diagnostics/reddit/YYYY/MM/DD/reddit-YYYYMMDDTHHMMSSZ.json`.
   Commit message:
   `diagnostic(reddit): add reddit-YYYYMMDDTHHMMSSZ`.
8. The diagnostic object must conform to that schema and contain the run ID when
   known, failing stage, stable error code, concise message, retryability, tool,
   fetched instruction SHA when known, intended or created snapshot path, last
   successful stage, exposed HTTP status when known, and `redacted = true`.
9. Sanitize diagnostics. Never store prompts, raw page bodies, response headers,
   stack traces, cookies, tokens, credentials, private repository data, personal
   data, or full raw tool output. Limit the message to the minimum needed to
   identify the failing boundary.
10. Do not recursively create another diagnostic if diagnostic creation fails.
    Notify Alex that both the primary operation and diagnostic write failed.
11. A failure notification must state: New York timestamp, run ID if known,
    failed stage, error code, last successful stage, whether a normal snapshot
    was written, diagnostic path/URL if written, and the smallest safe next
    action. Do not replace this with a generic success or no-update message.

### Source

- URLs:
  - `https://www.reddit.com/r/popular/?geo_filter=US`
  - `https://www.reddit.com/r/popular/rising/?geo_filter=US`
- Platform: `reddit`
- Access: Web Search/public pages.
- Capture at most 10 total items in configured URL order and native page order.
- Capture native rank, title, subreddit, post URL, displayed age/timestamp,
  score, and comment count only when exposed.
- Store the page kind (`popular` or `rising`) in native metrics.
- Treat `rising` as a native label, not calculated acceleration.
- Use `coverage = country` only when the US filter is confirmed; otherwise use
  `community` and record the limitation.
- Do not store comments, authors, profiles, or full post bodies.

### Unusual-signal gate

Notify only for:
- a new #1 item when an exposed score is at least 5,000 or exposed comments are
  at least 1,000;
- a directly comparable rise of at least five positions accompanied by an
  exposed score increase of at least 100%;
- a material source-health transition, first blocked/stale baseline, the sixth
  consecutive unhealthy run, or each multiple of 24 unhealthy runs;
- malformed JSONL, collision, upload failure, read-back mismatch, or any hard
  failure covered by the diagnostic contract.

Do not notify for ordinary list churn, a repeated item, a rising label alone, or
`validation_status = not_available`.
