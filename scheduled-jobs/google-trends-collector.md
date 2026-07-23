# Google Trends Collector

- Status: enabled
- Timing mode: `condition_watch`
- Timezone: `America/New_York`
- Canonical instruction:
  [`instructions/public-source-google-trends.md`](../instructions/public-source-google-trends.md)

## Schedule

```ical
BEGIN:VEVENT
DTSTART:20260720T030000
RRULE:FREQ=HOURLY;INTERVAL=4
END:VEVENT
```

## Prompt

```text
Run one Google Trends US source collection.

Before collecting data, fetch qobeat/social-trends@main file instructions/public-source-google-trends.md with the connected GitHub App and execute its Task instruction exactly. Never use a cached copy. If the canonical file cannot be fetched, notify Alex in Russian and do not collect or write a snapshot. Do not create, update, pause, replace, or disable Scheduled Tasks from inside the run. If collection and storage succeed without a canonical unusual-signal condition, emit no user-facing notification.
```
