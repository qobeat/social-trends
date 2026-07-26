# Hacker News Collector

- Status: enabled
- Timing mode: `condition_watch`
- Timezone: `America/New_York`
- Canonical instruction:
  [`instructions/public-source-hacker-news.md`](../instructions/public-source-hacker-news.md)

## Schedule

```ical
BEGIN:VEVENT
DTSTART:20260720T032000
RRULE:FREQ=HOURLY;INTERVAL=4
END:VEVENT
```

## Prompt

```text
Run one Hacker News source collection.

Before collecting data, fetch qobeat/social-trends@main file instructions/public-source-hacker-news.md with the connected GitHub App and execute its Task instruction exactly. Never use a cached copy. If the canonical file cannot be fetched, notify Alex in Russian and do not collect or write a snapshot. Do not create, update, pause, replace, or disable Scheduled Tasks from inside the run. If collection and storage succeed without a canonical unusual-signal condition, emit no user-facing notification.
```
