# Live Automations Inventory — {{PROJECT_NAME}}

> Snapshot {{SNAPSHOT_DATE}} (source: {{FILL_SOURCES}}). Refresh when stale or contradicted.

Operating reference for every deployed automation in this project. If an automation's prompt/config lives in an external system, its latest copy belongs in `proto/mirrors/` — this file is the summary, the mirror is the source of truth for behaviour.

## <Automation name>

- **Trigger / cadence:** <!-- TODO: what starts a run, how often -->
- **Runs as:** <!-- local scheduled task | remote cloud routine | Notion automation | script/cron -->
- **Behaviour:** <!-- TODO: input → transform → output, 3 lines max; details in the mirror -->
- **Guarantees:** <!-- TODO: read-only on X, idempotency/dedup key, output language, allowed writes -->
- **Logging:** <!-- TODO: where each run records its outcome; triage view -->
- **Mirror:** <!-- proto/mirrors/<file>, or "none (lives in this repo)" -->
- **Known issues / pending changes:** <!-- TODO: with task IDs -->

<!-- Repeat per automation. -->
