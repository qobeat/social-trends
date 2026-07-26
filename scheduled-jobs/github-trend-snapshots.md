# GitHub Trend Snapshots

- Status: enabled
- Timing mode: `condition_watch`
- Timezone: `America/New_York`
- Canonical instructions:
  [`instructions/github-trends-task.md`](../instructions/github-trends-task.md)
  and
  [`instructions/trend-collector-common.md`](../instructions/trend-collector-common.md)

## Schedule

```ical
BEGIN:VEVENT
DTSTART:20260720T033000
RRULE:FREQ=HOURLY;INTERVAL=4
END:VEVENT
```

## Prompt

```text
Run one GitHub trend source collection.

Use the existing task only; never create, update, pause, replace, or disable
Scheduled Tasks from inside the run.

Before collecting data:
1. Fetch `qobeat/social-trends@main` file
   `instructions/github-trends-task.md` with the connected GitHub App.
2. Record the fetched content SHA as `instruction_sha`.
3. Execute the file's `Task instruction` exactly. Never use a cached copy.

The canonical instruction must fetch and apply
`instructions/trend-collector-common.md` and governs the four-hour window,
repository-history baseline, public-data boundary, validation, immutable
storage, read-back, source health, and unusual-signal notification rules.

If either canonical file cannot be fetched, notify Alex in Russian and do not
collect or write a snapshot. If collection, storage, and read-back succeed
without a canonical unusual-signal condition, emit no user-facing notification.
```
