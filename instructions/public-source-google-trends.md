# Google Trends US Collector

## Task instruction

Run one lightweight raw trend-source collection for `google_trends_us`. Do not
produce a general news digest or sentiment analysis.

### Shared collector contract

Before collection, fetch `instructions/trend-collector-common.md` from
`qobeat/social-trends@main` and apply it as the governing shared contract.
Never use a cached copy. If it cannot be fetched, notify Alex in Russian and do
not collect or write a snapshot. Retain its fetched SHA as
`common_contract_sha` for run diagnostics. This file defines only the
source-specific additions and thresholds.
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

### Source-native context and domain classification

Classify context from the Google Trends response already loaded for collection. Do
not open Explore, Search it, news, article, or other pages merely to classify a
trend.

- For each captured trend, read at most the first three related queries exposed
  in its `Trend breakdown`.
- Store those queries as one compact `native_metrics.context_terms` string
  separated by ` | `; omit it when no related query is exposed.
- When the breakdown provides explicit evidence, store:
  - `native_metrics.domain`: exactly one of `Autos and Vehicles`,
    `Beauty and Fashion`, `Business and Finance`, `Climate`,
    `Entertainment`, `Food and Drink`, `Games`, `Health`,
    `Hobbies and Leisure`, `Jobs and Education`,
    `Law and Government`, `Other`, `Pets and Animals`, `Politics`,
    `Science`, `Shopping`, `Sports`, `Technology`, or
    `Travel and Transportation`;
  - `native_metrics.subdomain`: a short specific label such as
    `Film / Marvel`, only when directly supported;
  - `native_metrics.domain_confidence`: `high`, `medium`, or `unknown`;
  - `native_metrics.classification_source`:
    `google_trends_breakdown` or `unclassified`.
- Use `high` only when at least two related queries independently identify the
  same entity/domain. Use `medium` for one unambiguous related query. Otherwise
  use `unknown`, set `classification_source = unclassified`, and do not
  guess from an ambiguous top-level title.
- Preserve the exact top-level title in `trend.topic`. Set
  `trend.canonical_topic` to a specific normalized entity or event only when
  the breakdown explicitly identifies it; otherwise use the normalized
  top-level title.
- Example: `doomsday` plus `doomsday trailer` and
  `avengers doomsday trailer` becomes canonical topic
  `Avengers: Doomsday trailer`, domain `Entertainment`, subdomain
  `Film / Marvel`, confidence `high`.
- Domain classification describes context only. It is not sentiment,
  materiality, or an independent source signal.
- If a trend passes the unusual-signal gate but remains unclassified, at most
  one exact quoted Web Search may be used solely to explain that alert. Do not
  store external article content or externally inferred fields in the raw
  source snapshot. Cite the verification in the alert; if still ambiguous,
  report domain `unknown`.

A material alert must include the source-native domain/subdomain and canonical
topic when available.
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