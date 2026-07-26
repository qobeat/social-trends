# Reddit Trends Collector

- Status: enabled
- Timing mode: `condition_watch`
- Timezone: `America/New_York`
- Canonical instruction:
  [`instructions/public-source-reddit.md`](../instructions/public-source-reddit.md)

## Schedule

```ical
BEGIN:VEVENT
DTSTART:20260720T031000
RRULE:FREQ=HOURLY;INTERVAL=4
END:VEVENT
```

## Prompt

```text
Run one Reddit Popular US source collection.

Use the existing task only; never create, update, pause, replace, or disable Scheduled Tasks from inside the run.

At run start:
1. Set current UTC diagnostic timestamp, stage=`bootstrap`, and last_successful_stage=null.
2. Fetch `qobeat/social-trends@main` file `instructions/public-source-reddit.md` with the connected GitHub App.
3. Record the fetched content SHA as `instruction_sha` and execute that file's Task instruction exactly. Never use a cached copy.

Minimal failure wrapper:
- A blocked, empty, stale, or unparsable Reddit page is not an unhandled task failure. Follow the canonical instruction and write a schema-valid zero/partial snapshot with compact `source_status.message` diagnostics.
- On hard failure after canonical instruction fetch, follow the canonical diagnostic contract and attempt exactly one sanitized immutable diagnostic JSON under `data/diagnostics/reddit/YYYY/MM/DD/`.
- If the canonical instruction cannot be fetched, set stage=`instruction_fetch`, error_code=`canonical_instruction_fetch_failed`, instruction_sha=null, run_id=null when unknown, and attempt exactly one diagnostic JSON using `schemas/collector-diagnostic.schema.json` if GitHub is still usable.
- Diagnostics must be minimal and redacted: no raw tool output, prompts, page bodies, headers, stack traces, tokens, cookies, credentials, private data, or personal data.
- If diagnostic creation also fails, do not retry recursively. Notify Alex in Russian with both failures, failed stage, stable error code, last successful stage, and the smallest safe next action.

On any failure, provide a concrete Russian diagnostic with New York time, run ID when known, failed stage, error code, last successful stage, normal snapshot status, diagnostic path/URL if written, and retryability.

If collection, storage, and read-back succeed without a canonical unusual-signal condition, emit no user-facing notification.
```
