# Scheduled trend jobs

This directory records the exact prompts and schedules for the active ChatGPT
Scheduled Tasks that collect, correlate, or summarize the repository's trend
data.

These files are documentation snapshots, not executable scheduler
configuration. Editing them does not change a Scheduled Task. After changing a
task in ChatGPT, update the matching file here in the same change so the
repository remains an auditable copy of the deployed prompt.

Snapshot date: **2026-07-23**  
Default timezone: **America/New_York**

## Pipeline

| Stage | Job | Cadence | Primary repository dependency |
| --- | --- | --- | --- |
| Collect | [Google Trends Collector](google-trends-collector.md) | Every 4 hours | `instructions/public-source-google-trends.md` |
| Collect | [Reddit Trends Collector](reddit-trends-collector.md) | Every 4 hours | `instructions/public-source-reddit.md` |
| Collect | [Hacker News Collector](hacker-news-collector.md) | Every 4 hours | `instructions/public-source-hacker-news.md` |
| Collect | [GitHub Trend Snapshots](github-trend-snapshots.md) | Every 4 hours | `instructions/github-trends-task.md` |
| Correlate | [Cross-Trend Signals](cross-trend-signals.md) | Every 4 hours | `instructions/public-trends-task.md` |
| Alert | [Tactical Brief](tactical-brief.md) | Every 8 hours | Live Web Search |
| Summarize | [Today's Digest](todays-digest.md) | Daily at 11:00 ET | Confirmed snapshot history |

All seven jobs were enabled when this snapshot was created. Disabled and
superseded tasks are intentionally excluded.

## Update checklist

1. Read the current task in ChatGPT and copy its prompt verbatim.
2. Copy the full iCalendar schedule, timing mode, and timezone.
3. Preserve links to canonical repository instructions; do not duplicate those
   instructions inside a scheduler wrapper.
4. Check that every collector forbids changing Scheduled Tasks from inside a
   run and defines its failure/notification behavior.
5. Review the documentation diff before publishing.
6. Confirm that no task IDs, conversation IDs, tokens, cookies, credentials, or
   private connector output were committed.

## Source of truth

The deployed ChatGPT Scheduled Task is the source of truth for its wrapper
prompt and schedule. Files under `instructions/` are the source of truth for
collection logic. This split keeps scheduler wrappers small while allowing
canonical instructions, schemas, and validation rules to evolve under normal
repository review.
