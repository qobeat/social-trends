# ChatGPT Scheduled Task — Public Trend Snapshots v1

## Recommended task settings

- Title: `Public Trend Snapshots`
- Schedule: every hour
- Mode: monitoring/condition watch
- Output language: Russian
- Repository: `qobeat/social-trends`
- Branch: `main`

Paste the instruction below into a ChatGPT Scheduled Task.

---

## Task instruction

Run one public-trend collection iteration. This is a raw trend collection task,
not a sentiment or general-news task.

### Goal

Collect reproducible public trend observations available to a ChatGPT Scheduled
Task, save the complete raw snapshot as immutable JSONL in
`qobeat/social-trends`, compare it with prior runs when comparable evidence is
available, and notify Alex in Russian only about material changes or collection
failures requiring action.

### Non-goals

Do not analyze comments, sentiment, emotion, stance, private feeds, or individual
users. Do not scrape login-protected pages. Do not claim that indexed individual
posts represent platform-wide trends.

### Required tools

Use only:

1. **Web Search** to search and open public pages, public RSS, and public JSON.
2. Connected **GitHub App**:
   - `github_fetch_file` to read the schema and verify the created snapshot;
   - `github_create_file` to create a new immutable JSONL snapshot.

Never use `github_update_file` or `github_delete_file` under `data/`.
Never use browser login, cookies, API keys, shell, Python, unofficial proxy APIs,
or webhooks.

If either Web Search or the GitHub App is unavailable, report the blocker. Do not
claim that collection or upload succeeded.

### Time window

1. Get current UTC time and America/New_York time.
2. Set `window.end` to current UTC time.
3. Set `window.start` to exactly three hours earlier.
4. Set `lookback_hours` to `3`.
5. Set:
   - `run_id = public-YYYYMMDDTHHMMSSZ`;
   - `task_id = public-trends-v1`;
   - `collected_at = window.end`.
6. Use RFC 3339 UTC timestamps with `Z`.

A source with a broader native window may still be captured, but every record
must use `freshness = source_window_broader` and state the limitation. Do not
convert a daily/weekly source into a claimed three-hour metric.

### Schema and output path

Before collection, fetch:

- repository: `qobeat/social-trends`
- path: `schemas/trend-record.schema.json`
- ref: `main`

Every non-empty JSONL line must conform to schema version `1.0.0`.

Create exactly one new file per run:

`data/public/YYYY/MM/DD/public-YYYYMMDDTHHMMSSZ.jsonl`

The file is append-only at repository level: create it once and never modify it.

### Configured source registry

Attempt every source on every run. Capture at most 10 trend observations per
source.

#### 1. Google Trends US

- source_id: `google_trends_us`
- platform: `google_trends`
- URL: `https://trends.google.com/trending?geo=US`
- Method: Web Search/public page.
- Select United States, Past 4 hours, active trends where available.
- Prefer rank, trend title, search-volume bucket, growth, started time, and active
  status exposed by the page.
- Retain items started or updated within the three-hour window when timestamps
  permit.
- Search volume is attention, not sentiment.

#### 2. TikTok Creative Center

- source_id: `tiktok_creative_center_us`
- platform: `tiktok`
- URL: `https://ads.tiktok.com/creative/creativeCenter/trends`
- Method: Web Search/public page.
- Use United States and the shortest public time window available.
- Capture public hashtag/topic rank and native metrics only.
- Normally mark `freshness = source_window_broader`.
- This is a delayed/broader commercial trend signal, not a verified three-hour
  TikTok-wide snapshot.

#### 3. YouTube Charts US

- source_id: `youtube_charts_us`
- platform: `youtube`
- Discovery URL: `https://support.google.com/youtube/answer/7239739`
- Method: Web Search/public YouTube Chart pages.
- Use current official YouTube Charts for available US categories such as music,
  movie trailers, and gaming.
- Do not use the removed general YouTube Trending page.
- Store the chart category in `native_metrics.chart_category`.
- Set `coverage = category`; never label it platform-wide.

#### 4. Reddit Popular US

- source_id: `reddit_popular_us`
- platform: `reddit`
- URLs:
  - `https://www.reddit.com/r/popular/?geo_filter=US`
  - `https://www.reddit.com/r/popular/rising/?geo_filter=US`
- Method: Web Search/public page.
- Capture rank, title, subreddit, post URL, age/timestamp, score, and comment count
  only when exposed.
- Treat `rising` as a native source label, not a calculated acceleration rate.
- Set `coverage = country` when the US filter is confirmed; otherwise
  `coverage = community` and record the limitation.

#### 5. Hacker News

- source_id: `hacker_news_top`
- platform: `hacker_news`
- URLs:
  - `https://news.ycombinator.com/`
  - `https://news.ycombinator.com/best`
  - `https://hacker-news.firebaseio.com/v0/topstories.json`
- Method: public page/public JSON through Web Search.
- Capture rank, story ID, title, URL, points, comment count, and age/timestamp when
  exposed.
- Set `coverage = community`; Hacker News is a technology-community signal.

#### 6. Bluesky public trends

- source_id: `bluesky_public_trends`
- platform: `bluesky`
- URL:
  `https://public.api.bsky.app/xrpc/app.bsky.unspecced.getTrendingTopics?limit=10`
- Method: public JSON through Web Search.
- Capture topic, display name, link/query value, and native ordering.
- The endpoint is `unspecced`; record that in `limitations`.
- If direct JSON cannot be opened, search the exact endpoint and official Bluesky
  API references before marking the source blocked.

#### 7. Mastodon instance trends

- source_id: `mastodon_social_trends`
- platform: `mastodon`
- URLs:
  - `https://mastodon.social/api/v1/trends/tags?limit=10`
  - `https://mastodon.social/api/v1/trends/statuses?limit=10`
  - `https://mastodon.social/api/v1/trends/links?limit=10`
- Method: public JSON through Web Search.
- Capture the endpoint type in `native_metrics.endpoint_type`.
- Set `coverage = instance` and scope `mastodon.social instance view`.
- Never describe one instance's list as Fediverse-wide.

### Explicitly excluded in v1

Do not collect platform-wide Facebook, Instagram, Threads, X/Twitter, Pinterest,
Discord, or Telegram trends. They do not expose a reproducible unauthenticated
platform-wide trend feed usable by this Scheduled Task. Record no fabricated
placeholder observations for them.

### JSONL construction

Build records in this exact order:

1. One `run_start`.
2. All `trend_observation` records, grouped by configured source order and
   native rank.
3. Exactly one `source_status` per configured source, including failed sources.
4. One `run_end`.

Use record IDs:

- `<run_id>/run_start/000`
- `<run_id>/<source_id>/trend/NNN`
- `<run_id>/<source_id>/status/000`
- `<run_id>/run_end/000`

Within a run, remove exact duplicates from the same source by normalized
`item_url + topic`. Do not deduplicate across different runs; repeated snapshots
are required for time-series analysis.

Do not invent missing ranks, timestamps, metrics, IDs, or trend direction. Use
JSON `null`, an empty metrics object, `unknown`, or an explicit limitation.

Keep `raw_label` at 1,000 characters or less. Do not store comments, full
articles, private data, credentials, cookies, or unnecessary usernames.

### Trend state

Use `new`, `rising`, `falling`, `stable`, or `reentered` only if the
current source explicitly exposes that state or the task has a directly
comparable prior observation from the same source, scope, native window, and
metric. Otherwise use `unknown`.

Never calculate a cross-platform score in raw JSONL.

### Validation

1. Confirm every line is one compact JSON object with no Markdown fence.
2. Confirm every line parses independently.
3. Check required fields, enums, timestamp formats, record order, unique
   `record_id`, source-status completeness, and output-path pattern.
4. If an actual JSON Schema validator is available, validate each line and set
   `validation_status = passed` only on success.
5. If no validator is available, perform the structural checks and set
   `validation_status = not_available`; do not falsely claim schema validation.
6. If any JSON line is malformed, do not upload the file.

### GitHub upload

Use `github_create_file` with:

- repository: `qobeat/social-trends`
- branch: `main`
- path: the immutable output path
- message: `data(public): add <run_id> snapshot`
- content: the complete JSONL text with a final newline

If the path already exists, do not update or overwrite it. Report a run-ID
collision and stop.

After creation, use `github_fetch_file` to read the same path from `main`.
Verify that the first and last record IDs and total non-empty line count match
the prepared snapshot. Only then say that upload succeeded.

### User notification

Always create the raw snapshot when collection and GitHub write tools work.

Notify Alex in Russian only if:

- a new materially important trend appeared;
- a directly comparable source shows material rank/volume movement;
- the same topic appears in at least two independent sources;
- a source became blocked/stale after previously working;
- schema construction, GitHub write, or read-back verification failed.

If there is no material update and all storage checks pass, do not send a
notification.

A notification must include:

- New York timestamp and run ID;
- concise material changes;
- confirmed facts versus inference;
- source coverage limitations;
- GitHub snapshot path and commit/result confirmation;
- direct source URLs.

Never state that sentiment was measured in version 1.
