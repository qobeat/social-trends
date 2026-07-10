# social-trends

Append-only raw trend snapshots collected by ChatGPT Scheduled Tasks.

Version 1 deliberately collects **trends only**. Sentiment, comments, stance, and
emotion analysis are deferred to a later version.

## Tasks

- [Public/social trends task](instructions/public-trends-task.md)
- [GitHub trends task](instructions/github-trends-task.md)

Both tasks use only sources that a ChatGPT Scheduled Task can access through
public Web Search/pages or the connected GitHub App. A missing or blocked source
is recorded as a `source_status` JSONL record; the task must not fabricate or
silently substitute platform-wide data.

## Storage model

Each run creates one immutable JSONL file:

```text
data/public/YYYY/MM/DD/public-YYYYMMDDTHHMMSSZ.jsonl
data/github/YYYY/MM/DD/github-YYYYMMDDTHHMMSSZ.jsonl
```

Existing data files are never updated, deleted, or reused. This gives append-only
history at repository level and avoids concurrent GitHub file-update conflicts.

Each non-empty line must independently validate against
[`schemas/trend-record.schema.json`](schemas/trend-record.schema.json).

A run contains:

1. one `run_start` record;
2. zero or more `trend_observation` records;
3. one `source_status` record per configured source;
4. one `run_end` record.

## Version 1 sources

Public/social task:

- Google Trends — US Trending Now, strongest general trend source;
- TikTok Creative Center — delayed/broader-window TikTok trend signal;
- YouTube Charts — category-limited, not a platform-wide Trending page;
- Reddit r/popular and r/popular/rising — public page, best effort;
- Hacker News top/best stories — public page and Firebase API;
- Bluesky public trending-topics API — public but `unspecced`;
- Mastodon trends API on named instances — instance-scoped, not Fediverse-wide.

GitHub task:

- GitHub Trending/Explore through Web Search;
- connected GitHub App repository search and repository metadata;
- GitHub public repository pages for verification.

Facebook, Instagram, Threads, platform-wide X, Pinterest, Discord, and Telegram
are excluded from v1 because a Scheduled Task does not have a reproducible,
unauthenticated platform-wide trend feed for them. Individual indexed posts are
not sufficient evidence of a platform trend.

## GitHub write protocol

The tasks must use the connected GitHub App:

- `github_create_file` for each new immutable snapshot;
- `github_fetch_file` only for instructions/schema verification;
- never `github_update_file` or `github_delete_file` under `data/`.

Repository: `qobeat/social-trends`  
Branch: `main`

If GitHub write permission is unavailable, the task must report the blocker and
must not claim that data was uploaded.

## Data rules

- UTC timestamps in RFC 3339 format.
- US geography unless a source is global or instance-scoped.
- Preserve native rank/metric names; do not invent comparable cross-platform
  scores in raw data.
- Store short public trend labels and metadata, not comments or private data.
- Preserve repeated observations across runs: they form the time series.
- Deduplicate only within one source in one run.
- Record access failures, stale windows, and incomplete coverage explicitly.
- Never commit credentials, cookies, tokens, or private account identifiers.

## Similar projects reviewed

- `antonkomarev/github-trending-archive`: useful precedent for scheduled,
  immutable historical snapshots and minimal raw data.
- `igrigorik/gharchive.org`: strong precedent for preserving hourly public
  GitHub events before deriving trends.
- `GoogleTrends/data`: separates normalized public datasets from analysis.
- `ericciarla/trendFinder`: separates collection, analysis, and notifications,
  but relies on paid/API-key services.
- `JustAnotherArchivist/snscrape`: broad scraper design, but upstream changes
  have broken major platform collectors.
- `GeneralMills/pytrends`: useful historical interface but archived in 2025.
- `huchenme/github-trending-api`: useful schema precedent; hosted API has
  longstanding availability issues.

The v1 design therefore favors public first-party sources, explicit provenance,
immutable snapshots, and source-failure records over scraping hidden endpoints.
