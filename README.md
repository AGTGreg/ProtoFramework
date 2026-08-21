# ProtoFramework

Greg's project-workspace framework: the session-persistence protocols (WORKLOG, repo-local memory, mirrors, decisions, context snapshots), packaged as a versioned template that the **proto-init** skill (plugin `proto`) copies into any new or existing project — personal or company, any project type. Project- and organization-specific details (stack, conventions, connections, people) enter only at init time or while working on the project; the template itself carries none.

## How it's consumed

Humans don't copy this by hand — `proto-init` does:

1. Clones this repo to a temp dir (or uses a local checkout), strips `.git`.
2. Reads `manifest.json` — the machine-readable copy plan (files, destinations, merge strategies).
3. Copies `core/` + the modules the user confirms (à la carte, proposed from each module's `offer` line — no bundles), fills `{{PLACEHOLDERS}}`, runs the codebase scan, auto-fills from the external sources the user names in the interview (task tracker, knowledge base) when their tools are available.
4. Stamps the version into the target repo as `proto/VERSION`.

## Versioning

- `PROTO_VERSION` — semver of this template.
- Protocol text in `core/CLAUDE.md` is wrapped in markers: `<!-- proto:begin <block>@<version> -->` … `<!-- proto:end <block> -->`. The `proto-update` skill swaps outdated blocks in initialized projects without touching project-specific text.
- Every change: bump versions (file + affected block markers), record in `CHANGELOG.md`.

## Update policy

- **MINOR/PATCH** releases change protocol text inside blocks and/or add new blocks or files — proto-update applies them mechanically to initialized projects.
- Anything that **moves, renames, or deletes files in target projects is a MAJOR release**; its CHANGELOG entry must carry an explicit migration list. proto-update stops at major boundaries and walks the migration with the owner.
- **Removing a block** = tombstone entry in `manifest.json` → `removed_blocks` (name, removed_in, disposition). Tombstones are permanent, so skip-version updates stay correct.
- **Data files** (STATE, WORKLOG, decisions, IDEAS, connections, memory) **and module files are never updated in place** — they are project data from the moment they're copied. All updatable protocol lives in CLAUDE.md blocks; data files carry only a pointer line to their block.

## Editing rules

- Protocol text changes go **inside** the marked blocks; bump the block's `@version` and `PROTO_VERSION`.
- Placeholders are `{{UPPER_SNAKE}}` and must be listed in `manifest.json`.
- New files must be added to `manifest.json` (core or a module with an `offer` line) or proto-init will ignore them.
- Org-specific content (company names, team members, standard stacks, delivery workflows, tool URLs) must **never** be baked into the template — it enters via placeholders/TODOs filled at init or during project work.

## Layout

```
core/        — every project gets these (CLAUDE.md protocols, STATE position file, IDEAS parking lot, WORKLOG, memory/, decisions.md, connections, archives/)
modules/     — à-la-carte templates, proposed individually at init (architecture, environments, client, automations-inventory, working-docs)

In target projects, everything lands inside a `proto/` directory — the operational
record, cleanly separated from the project's own files. Only CLAUDE.md stays at the
root (it must, for auto-loading).
manifest.json  — copy plan; blocks + placeholders registry; per-module `offer` lines
PROTO_VERSION  — template semver
```
