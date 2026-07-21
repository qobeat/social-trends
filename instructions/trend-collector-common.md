# Trend Collector Common Contract

## Shared task instruction

This file is the authoritative shared contract for all raw trend collectors in
`qobeat/social-trends`. A source-specific instruction may add source URLs,
allowed source tools, selection rules, mappings, diagnostics, and unusual-signal
thresholds, but must not weaken or contradict this contract.

### Repository and cadence

- Repository: `qobeat/social-trends`.
- Branch: `main`.
- Cadence and comparison window: four hours.
- Set `window.end` to current UTC, `window.start` to exactly four hours earlier,
  and `lookback_hours = 4`.
- Obtain and retain the current America/New_York time for any notification.

### Canonical inputs and prior baseline

1. Fetch the source-specific instruction and this common contract fresh from
   `main`; never use a cached copy.
2. Fetch `schemas/trend-record.schema.json` fresh from `main` before building
   output.
3. Use `github_search_commits` with the source-specific commit-message prefix
   and fetch the latest confirmed same-collector snapshot from `main`.
4. Repository history, not conversation memory, is the prior-state authority.
5. Compare only the same source ID, native window/query identity, and directly
   comparable native metric. Otherwise use `trend_state = unknown`.

### Collection and public-data boundary

- Use only tools explicitly allowed by the source-specific instruction.
- Preserve native source order. Never create a synthetic cross-query rank.
- Capture only metadata actually exposed by the configured public source.
- Never store prompts, raw tool output, headers, cookies, credentials, personal
  data, private-repository data, or private project context.
- Never store an exact rendered personalized search query in public JSONL.
  Store stable generic query IDs and a versioned public query-contract ID
  instead.
- A successful empty result is `no_data`; a source access failure is `blocked`;
  a partial source/query failure is `partial`; exact repeated source output may
  be `stale` when the source-specific instruction defines comparability.

### JSONL, validation, and immutable storage

- Use schema version `1.0.0` and compact one-object-per-line JSONL.
- Order: one `run_start`; observations in source-defined order; exactly one
  `source_status` per configured source; one `run_end`.
- Use unique IDs:
  `<run_id>/run_start/000`,
  `<run_id>/<source_id>/trend/NNN`,
  `<run_id>/<source_id>/status/000`, and
  `<run_id>/run_end/000`.
- The source-specific instruction declares the run-ID prefix, task ID, immutable
  output path, and commit-message prefix.
- Never overwrite, update, or delete a file under `data/`.
- Validate independent JSON parsing, required fields, enums, UTC timestamps,
  path, record order, unique IDs, one status per source, non-empty line count,
  and final newline.
- If an actual JSON Schema validator is available, validate every line and use
  `validation_status = passed`; otherwise perform all structural checks and use
  `validation_status = not_available`.
- Create exactly one immutable snapshot with `github_create_file`.
- Fetch it from `main` and verify exact content, first and last record IDs,
  non-empty line count, and final newline. Success requires this read-back.

### Health and notification

- Maintain `consecutive_unhealthy_runs` from the immediately prior confirmed
  snapshot: increment for continued `blocked`/`stale`, set to 1 for a new
  unhealthy state, and reset to 0 for a healthy state.
- Save a valid snapshot whenever collection and storage remain possible,
  including schema-valid zero/partial source outcomes.
- Notify in Russian only for a source-specific unusual signal, a material health
  transition/recovery, the first unhealthy baseline, the sixth consecutive
  unhealthy run, each multiple of 24 unhealthy runs, or validation/storage/
  read-back failure.
- Do not notify for ordinary turnover, unchanged limitations, or
  `validation_status = not_available` after successful structural checks.
- If collection, storage, and read-back succeed without a material condition,
  emit no user-facing notification.

### Conflict rule

If a source-specific instruction conflicts with this file on repository, branch,
four-hour window, prior-state authority, public-data boundary, validation,
immutability, read-back, or notification economy, this common contract governs.
