---
name: proto-init
description: Initialize a new or existing project repo with Greg's ProtoFramework (WORKLOG/memory/mirrors/decisions protocols, context snapshots, plus à-la-carte context modules), auto-filling from the connected sources the user names during the interview. Works for any project type — software, automation, research, content, ops. Use instead of /init on projects that adopt the ProtoFramework. Trigger on "proto-init", "proto init", "initialize this project with proto", "set up the project framework", "scaffold the workspace protocols".
---

# proto-init

Copies the versioned ProtoFramework template into the current project and customizes it. Works on empty dirs, fresh repos, and long-lived codebases. Never overwrites existing content, never adds a remote, never pushes.

## Template source (in priority order)

1. `PROTO_TEMPLATE_PATH` environment variable set → use that local checkout directly (do NOT copy its `.git`).
2. Otherwise clone `https://github.com/AGTGreg/ProtoFramework` to a temp dir, strip `.git`.
3. Clone fails (offline, no git) → say so and ask the user for a local path.

Read `manifest.json` from the template root. It defines: the file copy plan (`core` + à-la-carte `modules`, each with an `offer` line saying when to propose it), per-file `strategy` (`merge-blocks` | `copy-if-absent`), the marked-block names, and the placeholder registry. The manifest is the authority — never hardcode the file or module list.

## Procedure (deterministic — follow in order)

### 1. Preflight

- `git rev-parse --is-inside-work-tree` — not a repo → ask: "Not a git repo. Run `git init -b main`?" (do it on yes; abort on no).
- `git status --porcelain` — dirty tree → STOP: ask the user to commit or stash first. Never init over uncommitted work.
- `proto/VERSION` file exists in the target → ABORT: "Already proto-initialized (vX). To bring the protocols up to date, run proto-update instead."

### 2. Interview 1 — shape (ask as ONE grouped question set)

1. Project name (default: directory name).
2. Project type — free text, whatever the user says (software dev, automation, research, content, ops, mixed…). Not an enum; it steers module proposals in step 2b and tag suggestions.
3. WORKLOG tag slugs — propose 4-8 kebab-case workstream slugs from what you know of the project; user edits. Always include `workspace`.
4. Roadmap stages, if any? (free text, e.g. "explore → build → harden"; enter to skip → the STATE.md roadmap section says "no stages")
5. Local-only repo? (yes → add "commit freely, never add a remote or push" to the `{{HARD_RULES}}` list)
6. Auto-fill sources: "Which external systems hold state for this project? (task tracker URL, knowledge base / wiki, none)" — whatever the user names drives step 3 and seeds `proto/connections.md`.

### 2b. Module proposal (à la carte — no bundles)

Walk the manifest `modules` list. Propose each module whose `offer` line matches the project (type from the interview, presence of source code, deploy configs, client mentions). Present the proposed set in ONE message — user adds/removes; zero modules is valid. Module answers drive module-specific follow-ups:

- `architecture` selected → ask baseline stack, if any (free text, e.g. "Python 3.12 · Django 5 · Postgres"; enter to skip → `{{STACK_BASELINE}}` becomes "none set" and the deviations section is dropped).
- `client` selected → client placeholders matter in Interview 2.

### 3. Auto-fill attempt (graceful — never block on it)

1. For each source the user named in interview question 6, ToolSearch for matching MCP tools (e.g. a task-tracker fetch tool, wiki find/read tools). No tool found for a source → note it in `proto/connections.md` as `not yet connected`, move on silently.
2. Task tracker URL given and fetchable → pull name, status, type, client, owners, open tasks — supplies *current state* (fills context overview, backlog, client contacts, commercial fields).
3. Knowledge base / wiki available → look up the project's entry by name/slug and read it — supplies *narrative* (fills overview history).
4. Record what was fetched. Every auto-filled file gets its `{{FILL_SOURCES}}` stamp set accordingly (e.g. "tracker + wiki", "interview").
5. Each confirmed source becomes a row in `proto/connections.md` (domain, tool, mechanism, notes).
6. Any fetch error → tell the user in one line, continue with interview.

### 4. Copy per manifest

For each entry in `core` + the selected modules:

- **`copy-if-absent`**: destination exists → skip, record in report. Absent → copy (create parent dirs).
- **`merge-blocks`** (CLAUDE.md): destination absent → copy whole file. Exists → keep ALL existing content as-is, then append ONLY the `<!-- proto:begin … -->…<!-- proto:end … -->` blocks from the template that are not already present (match by block name anywhere in the file), in template order, at the end of the file. Also append the non-block sections the blocks depend on if no equivalent exists: "Worklog tags", "Convention pointers", "How to work with the owner". Never duplicate a block; never modify existing lines.
- After a merge into an existing CLAUDE.md: if the pre-existing content includes project knowledge rather than rules of operation (current status, environment details, backlogs, architecture notes), leave it untouched but list each such section in the report with a suggested new home (`proto/STATE.md`, `proto/context/`, `proto/memory/`) — relocation is the owner's call, never automatic.

### 5. Codebase scan (skip for empty/new projects)

If the repo contains source code (if it does and the `architecture` module wasn't selected, re-offer it once): detect languages, frameworks and versions (lockfiles, configs), top-level layout, run/test/build commands (Makefile, package.json, manage.py, docker-compose, README), external services (API clients, env var names — names only, NEVER values). Write findings into `proto/context/architecture.md` under its SCAN markers, and add a short "Working on the code" section to CLAUDE.md (outside proto blocks) holding only rules of operation: build/test/run commands and a one-line stack pointer. Stack detail, layout, and external services go to `proto/context/architecture.md` only — CLAUDE.md stays rules-only. If a baseline stack was given in the interview, note deviations from it — ask the user *why* for each deviation and record the answer, or mark TODO. No baseline → record actuals only.

### 6. Interview 2 — gaps

Walk the remaining unfilled placeholders and TODO markers relevant to the selected modules (client contacts, commercial state, environments, connections, convention pointers). `{{CONVENTION_POINTERS}}` = where shared conventions live for this project (skills to invoke, wiki pages, docs) — "None yet" if the user has nothing. Ask in ONE grouped message, every item skippable. Skipped → leave the explicit `<!-- TODO -->` marker in place.

### 7. Finish

1. Replace all `{{PLACEHOLDER}}` tokens everywhere (unanswered → sensible empty defaults, but structural TODOs stay visible). `{{SNAPSHOT_DATE}}` = today. `{{PROTO_FRAMEWORK_VERSION}}` = template's PROTO_VERSION.
2. Write the template's `PROTO_VERSION` value into the target's `proto/VERSION`.
3. The copied WORKLOG.md's first entry: fill topic, modules, fill-sources; `{{INIT_TODOS}}` = the list of skipped/TODO items, or "None". `{{MODULES}}` = comma-separated selected module names, or "none".
3b. Fill the copied STATE.md: As-of = today, Goal = the one-liner, Roadmap position from the interview (or "no stages"), Milestone = the current stage's must-land items from what's known (or "owner to define"), Next = the project's first real action (or "owner to define"), Waiting on = any blocking skipped items.
4. `git add` exactly the files this run created/modified; commit: `Initialize project workspace with ProtoFramework vX.Y.Z (modules: <list or none>)`.
5. Print the report: **created** / **merged** (blocks appended) / **skipped collisions** / **skipped by condition** / **TODOs remaining** / **auto-fill sources used**. Remind: "Protocols are live from the next session onward; this session should already follow them."

## Guardrails (absolute)

- NEVER overwrite or rewrite existing file content — append-only merges, copy-if-absent otherwise.
- NEVER `git remote add`, NEVER push, NEVER touch anything outside the target repo (except reading the template).
- Secrets: record credential *locations*, never values.
- Nothing org-specific is baked into the template — every company/team/tool specific detail enters through the interview or auto-fill, or stays a TODO.
- If anything in the template contradicts the project's existing CLAUDE.md rules, the existing rules win — flag the conflict in the report instead of resolving it silently.
