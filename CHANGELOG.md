# Changelog

## 1.9.1 — 2026-08-21

proto-update hardened by a live run against a real 1.6.0-era project:

- Preflight accepts the legacy root `PROTO_VERSION` stamp (older installs) instead of aborting as "not initialized"; apply migrates the stamp to `proto/VERSION` and deletes the root file.
- Report now flags — never touches — legacy leftovers outside the markers: non-block sections superseded by blocks, and old-style data-file headers still carrying protocol text, each with its suggested manual cleanup.


## 1.9.0 — 2026-08-21

- **New skill: proto-update.** Brings an initialized project's blocks up to the template: plan (swap outdated / add missing + dependent sections / remove tombstoned / skip current-or-ahead) → owner confirms → apply marker-to-marker → stamp `proto/VERSION` → commit + per-block report. Stops at MAJOR boundaries and walks the CHANGELOG migration list with the owner; never downgrades; never touches anything outside block markers or any `proto/` data file.
- proto-init's already-initialized abort now points to proto-update.


## 1.8.0 — 2026-08-21

Updatability foundations: one home for protocol text, framework-owned blocks, tombstones, update policy.

- **All protocol text now lives in CLAUDE.md blocks — data files carry a one-line pointer.** New `decisions-protocol` block (numbering, freeze rules, entry format, rotation) and `connections-protocol` block (Mechanism values, Verified rule, credential locations); `ideas-protocol` (→1.8.0) absorbed the entry format. decisions.md, IDEAS.md, WORKLOG.md, STATE.md, connections.md headers slimmed to pointers — a block update can no longer contradict a frozen file header.
- **Ownership rule** in the map block (→1.8.0): proto blocks are framework-owned — never edit inside the markers; customizations live outside, and outside text wins on conflict.
- **Block tombstones**: `manifest.json` → `removed_blocks` records every deleted block (starting with `external-conventions`, removed in 1.7.0) so proto-update can clean them up across skipped versions.
- **Update policy** in the README: MINOR/PATCH = block swaps and additive changes only; file moves/renames/deletes = MAJOR with a written migration list; data files and module files are never updated in place.
- Blocks: 8 → 10.


## 1.7.1 — 2026-08-21

- proto-init template source is no longer a hardcoded personal path: `PROTO_TEMPLATE_PATH` env var → local checkout; otherwise clone the public repo (https://github.com/AGTGreg/ProtoFramework); otherwise ask. Closes the last machine-specific detail (audit finding N1) — the skill now works on any machine.


## 1.7.0 — 2026-07-24

Simplification pass on the protocol surface (audit findings N2-N5).

- Merged "Where things live" into the `change-discipline` block (→1.7.0) — now one version-managed map: Path | Holds | How it changes. Three overlapping descriptions of the filesystem became one.
- Removed the vestigial `external-conventions` block (blocks 9 → 8) — its principle line ("pointers, never copies") folded into the "Convention pointers" section header.
- Session-close checklist regrouped 9 → 5 items (→1.7.0), no content dropped: worklog+Focus, STATE, record-kept (decisions/ideas/memory), externals-flagged (mirrors/context), committed.
- Worklog tags rule no longer assumes a task tracker exists (`worklog-protocol` →1.7.0).


## 1.6.0 — 2026-07-17

- `orient` (→1.6.0): context snapshots are skimmed, not read — title + snapshot stamp per file at session start; full read only when a file bears on the current task. Mirrors the memory-index pattern; keeps orient cost flat as `proto/context/` grows. Stale/contradicted stamps still trigger a refresh of that file.

## 1.5.0 — 2026-07-17

Determinism audit: ambiguities and contradictions in the agent-facing protocol text resolved.

- **Project data moved out of versioned blocks** — `{{WORKLOG_TAGS}}` and `{{CONVENTION_POINTERS}}` now live in dedicated non-block sections ("Worklog tags", "Convention pointers") that proto-update never touches; the blocks reference them by name. Fixes future block updates clobbering project data.
- memory-protocol: qualification test now points to "the knowledge sources in `proto/connections.md`" instead of a leftover "the wiki".
- Focus rule de-circularized: a direct owner instruction is its own authorization (state milestone impact in one line, proceed); the challenge-and-park step applies to agent-proposed work.
- Stage transitions are proposals: all milestone items checked → agent drafts next Milestone + IDEAS sweep (promote/reject per idea), owner confirms, then STATE updates. Same authority rule added to the ideas sweep.
- Worklog entry format gains an optional `**Focus:**` line — required exactly when a session didn't advance the Milestone; checklist references it. `grep "Focus:"` doubles as a drift report.
- CLAUDE.md rules-only rule got its exemption defined: build/test/run commands ARE rules of operation; stack detail/layout/services go to `proto/context/architecture.md` only (scan step aligned).
- decisions.md flipped to newest-first under the `---` marker — same order as the worklog; rotation cuts from the bottom.
- Worklog timestamps declared local time; "owner" defined ("the human running this project"); merge-blocks now also appends the non-block sections blocks depend on.
- Blocks bumped to 1.5.0: state-protocol, ideas-protocol, worklog-protocol, memory-protocol, session-close-checklist, change-discipline, external-conventions. Unchanged: orient, mirrors-lifecycle (1.4.0).

## 1.4.0 — 2026-07-17

Namespaced installation: everything lands inside `proto/`.

- All proto files in target projects now live under a single `proto/` directory — STATE, IDEAS, WORKLOG, decisions, connections, memory/, context/, docs/, archives/, on-demand mirrors/, and the version stamp (now `proto/VERSION`). Only CLAUDE.md stays at the repo root, for its auto-load contract.
- Why: the project's own files own the root; one namespaced dir keeps the operational record visually separate and eliminates collisions with common project dir names (`docs/`, `context/`, `memory/`, `archives/`).
- All blocks except `external-conventions` bumped to 1.4.0 (path references); grep/rotation commands are repo-root-relative under `proto/`.
- proto-init: preflight checks `proto/VERSION`; scan writes to `proto/context/architecture.md`; merge report suggests `proto/…` homes.

## 1.3.0 — 2026-07-17

Focus discipline: milestones, an idea parking lot, and a lean-CLAUDE.md guard.

- STATE.md gains a **Milestone** section — the current stage's finish line as 3-7 must-land checkable items, nothing optional. All checked → stage advances, next milestone written, IDEAS.md swept. (`state-protocol` →1.3.0.)
- **Focus rule**: every task is checked against the Milestone before starting; non-advancing work is called out and proposed for parking — proceeding needs the owner's explicit "do it anyway".
- New core `IDEAS.md` + `ideas-protocol` block — cheap mid-session parking for out-of-scope ideas (2-4 lines each); curated, not append-only: every entry is eventually promoted into the Milestone/tracker or deleted; swept at milestone completion, not read every session.
- Session-close checklist (→1.3.0): "did this session advance the Milestone?" and "ideas parked?" items added.
- `change-discipline` (→1.3.0): CLAUDE.md row added — rules of operation only, project knowledge belongs in STATE.md/context/memory.
- proto-init: fills the initial Milestone; after merging blocks into an existing CLAUDE.md, flags pre-existing project-knowledge sections with suggested new homes (owner decides — never moved automatically).

## 1.2.0 — 2026-07-11

Session-continuity upgrades: a position file, accountable decisions, and a per-file discipline map.

- New core `STATE.md` — one-screen position file (Next / Goal / Roadmap position / Standing / Waiting on); rewrite-in-place, ~60-line cap, stale-STATE-is-a-bug rule. New `state-protocol` block; `orient` (→1.2.0) starts there.
- Decisions log upgraded: permanent `DEC-###` numbers (highest found by grep across `archives/decisions-*.md` — survives rotation), entries frozen except **Status** and **Looking back** (result filled in once known), replacement chain for changed minds, worklog-style monthly rotation.
- New `change-discipline` block — one table stating how every file may change (rewrite / append / curate / sync / write-once).
- `worklog-protocol` (→1.2.0): entries name the decisions they enact by `DEC-###`.
- `memory-protocol` (→1.2.0): week-to-week state routes to STATE.md as well as context/worklog.
- Session-close checklist (→1.2.0): NOW refresh + decision logging and Looking-back backfill added.
- `connections.md`: per-row **Verified** date — an unverified row is a claim, not a fact.
- New `working-docs` module (`docs/`) for long-form working material: investigation write-ups, playbooks, plans, hand-off specs.
- proto-init: roadmap-stages interview question; fills STATE.md at init.
- Simplification pass — same capability, fewer moving parts:
  - Flattened `decisions/log.md` → `decisions.md` — one file needs no folder; matches `connections.md` at root.
  - Removed the `automation-spec` module and with it the `templates/` dir — the framework ships protocols, not document templates; the automations inventory stands alone.
  - `mirrors/` is no longer scaffolded at init — created on demand with the project's first mirror, along with a `register.md` table (`mirrors-lifecycle` →1.2.0).
  - Worklog entries dropped their **Next:** line — STATE.md's Next section is the single home of the forward pointer.
  - Dropped the `Project tracker:` header line from CLAUDE.md (connections.md owns URLs) and merged `LOCAL_ONLY_RULE` into `HARD_RULES` — placeholders 14 → 12.
  - Dropped the Auth column from connections.md; Notes absorbs it.
