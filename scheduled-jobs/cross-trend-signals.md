# Cross-Trend Signals

- Status: enabled
- Timing mode: `condition_watch`
- Timezone: `America/New_York`
- Canonical instruction:
  [`instructions/public-trends-task.md`](../instructions/public-trends-task.md)

## Schedule

```ical
BEGIN:VEVENT
DTSTART:20260720T035000
RRULE:FREQ=HOURLY;INTERVAL=4
END:VEVENT
```

## Prompt

```text
Run one cross-trend correlation pass.

Before reading trend data, fetch qobeat/social-trends@main file instructions/public-trends-task.md with the connected GitHub App and execute its Task instruction exactly. Never use a cached copy. If the canonical file cannot be fetched, notify Alex in Russian and stop. Do not collect replacement source data, create repository snapshots, or create, update, pause, replace, or disable Scheduled Tasks from inside the run. If the canonical material gate is not met, emit no user-facing notification.
```
