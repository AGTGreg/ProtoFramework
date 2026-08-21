# {{PROJECT_NAME}}

{{PROJECT_ONE_LINER}}

<!-- proto:begin orient@1.6.0 -->
## Orient yourself (every session)

The `proto/` directory is the project's operational record — everything in it belongs to the working protocol, not the codebase.

0. Read `proto/STATE.md` — the project's position: first action, goal, roadmap stage, blockers.
1. Read `proto/WORKLOG.md` (top entry only) and `proto/memory/MEMORY.md` — what the last session did and the durable repo facts.
2. Skim `proto/context/` — for each file, read only its title and snapshot stamp. Full-read a file only when it bears on the current task. A stamp that is stale or contradicted by an external source of truth → refresh that file as part of your work.
3. If `proto/mirrors/` exists, check it — copies of artifacts that live in external systems; the worklog's Open state tells you if any are mid-deployment.
4. For anything deeper (clients, strategy, other projects), query the knowledge sources listed in `proto/connections.md`.
<!-- proto:end orient -->

<!-- proto:begin state-protocol@1.5.0 -->
## STATE.md protocol (the position file)

`proto/STATE.md` answers, on one screen, "where does this project stand and what do I do first."

- **Rewrite in place — it never grows.** Narrative belongs in `proto/WORKLOG.md`, reasoning in `proto/decisions.md`, durable facts in `proto/memory/`. Overflow past ~60 lines means something belongs elsewhere.
- The **Milestone** section is the current stage's finish line: 3-7 must-land items, checked off as they land. Nothing optional lives there.
- **Focus rule — before starting any task, check it against the Milestone.** For work the agent proposes or infers: if it doesn't advance a milestone item, say so and propose parking it in `proto/IDEAS.md`; proceed only on the owner's explicit "do it anyway". A direct owner instruction is its own authorization — state the milestone impact in one line and proceed.
- All milestone items checked → propose the stage transition to the owner: a draft of the next Milestone, the Roadmap position update, and the `proto/IDEAS.md` sweep (per idea: promote or delete — owner decides). Update STATE only after the owner confirms.
- Refresh STATE before ending any session that moved the project (see the session-close checklist).
- Honour the roadmap position: don't start later-stage work without the owner's explicit go-ahead.
- A stale `STATE.md` is a bug — correcting it precedes all other work.
<!-- proto:end state-protocol -->

<!-- proto:begin ideas-protocol@1.8.0 -->
## IDEAS.md protocol (the parking lot)

- An idea that surfaces mid-task and is out of scope → add a 2-4 line entry to `proto/IDEAS.md` and keep working. Capture must stay this cheap, or it won't happen.
- Entry format: `## <the idea, one line> (YYYY-MM-DD)` plus 1-2 lines — why parked, where it came from (worklog date / `DEC-###`).
- Curated, not append-only: every entry is eventually **promoted** (into the Milestone or the task tracker — then deleted here) or **rejected** (deleted, one-line worklog note).
- Reviewed at every milestone completion — not every session. Parked ideas resurface when re-planning, not as daily noise.
- The sweep is a proposal: the agent recommends promote/reject per idea; the owner decides.
<!-- proto:end ideas-protocol -->

## Hard rules

{{HARD_RULES}}

## Worklog tags

Project data, not protocol — proto-update never touches this section.

{{WORKLOG_TAGS}}

<!-- proto:begin worklog-protocol@1.7.0 -->
## WORKLOG.md protocol (deterministic — follow exactly)

`proto/WORKLOG.md` is the session-handoff journal. Newest entry first, directly under the `---` marker.

**READ:**
- At session start: read only the newest entry (top of file).
- Before touching any task or workstream: `grep -n "<tag>" proto/WORKLOG.md proto/archives/worklog-*.md` for its tags and read the matching entries. Never rely on the top entry alone when resuming older work.

**WRITE — append an entry at each of these moments, no exceptions:**
1. A unit of work is committed (one entry per commit or coherent commit group).
2. Work stops half-finished — blocked, deferred, or session ending mid-task.
3. A session ends after doing anything beyond reading.
Pure Q&A/reading sessions write nothing.

**Entry format (exact):**

```markdown
## YYYY-MM-DD HH:MM — <topic, one line> [tags: <task-ids and slugs>]
**Done:** what changed, with file paths.
**Open state:** half-finished / pending / awaiting-someone items. "None" if clean.
**Focus:** <only when the session did NOT advance the Milestone — one line on why>
**Commits:** <short hashes, or "none">
```

Timestamps are local time. The next action lives in `proto/STATE.md` → Next, not here — entries record what happened, STATE points forward.

- Tags: task-tracker IDs (`#123`, if the project has a tracker) plus slugs from the fixed list in this file's "Worklog tags" section (outside the proto blocks — it is project data, not protocol). Add a new slug only when a new workstream is born, and add it to that section in the same commit.
- Name any decision an entry enacts, informs, or contradicts by its `DEC-###` number.
- Max 12 lines per entry. Details belong in commit messages and files, not the worklog.
- **Open state MUST name any mirror in `proto/mirrors/` edited but not yet deployed to the live system, and any change not yet verified.** That is the single most important line in the file.

**ROTATION — hard cap 500 lines:**
- After appending, run `wc -l proto/WORKLOG.md`. If > 500: cut the oldest entries (keep at least the 10 newest) into `proto/archives/worklog-YYYY-MM.md` (append if the month's file exists), so WORKLOG.md drops under 300 lines. Do it in the same commit as the entry that breached the cap.
- Archived entries keep their exact format — the grep in READ covers `proto/archives/` so nothing becomes unsearchable.
<!-- proto:end worklog-protocol -->

<!-- proto:begin decisions-protocol@1.8.0 -->
## Decisions protocol (proto/decisions.md)

Record of meaningful decisions and why — capture the *why*, not just the *what*. Newest entry first, directly under the `---` marker, same order as the worklog.

- Every entry takes the next free `DEC-###` number. Numbers are permanent and survive rotation — find the highest with `grep -h "^## DEC-" proto/decisions.md proto/archives/decisions-*.md | sort -V | tail -1`.
- Past entries are frozen, with two exceptions: the **Status** line (when a decision is later replaced or withdrawn) and the **Looking back** line (filled once the real-world result is known). A change of mind is a fresh entry naming the one it replaces.
- Rotation: same scheme as the worklog — a new entry pushes the file past 500 lines → move the oldest entries (bottom of the file; keep at least the 10 newest) into `proto/archives/decisions-YYYY-MM.md`, in the same commit.

**Entry format (exact):**

```markdown
## DEC-001 — Short title (YYYY-MM-DD)
**Status:** standing | replaced by DEC-### | withdrawn
**Decision:** what was decided.
**Why:** the reasoning, constraints, and what would change our mind.
**Alternatives considered:** what else was on the table.
**Owner:** who's accountable.
**Looking back:** _blank at decision time — fill once the outcome is known._
```
<!-- proto:end decisions-protocol -->

<!-- proto:begin memory-protocol@1.5.0 -->
## Repo-local memory protocol (deterministic — follow exactly)

Durable facts live in `proto/memory/` **inside this repo** — never in the global `~/.claude` auto-memory (it must stay empty of this project's content).

**READ:**
- At session start (orient step 1): read `proto/memory/MEMORY.md` in full — it is the index, one line per fact.
- Read a fact file's body only when its index line is relevant to the current task. Write index lines so that this decision is makeable from the line alone.

**WRITE — a fact qualifies only if ALL three are yes:**
1. Will it matter in a future session? (Session narrative → `proto/WORKLOG.md`; one-off details → commit message.)
2. Is it NOT already derivable from repo files, git history, or the knowledge sources in `proto/connections.md`? (No duplication — the repo documents itself.)
3. Is it stable? (State that changes week-to-week → `proto/STATE.md`, `proto/context/`, or the worklog, not memory.)

**WRITE — triggers, all in the same commit as the related work:**
1. New qualifying fact learned (an instruction from the owner, a discovered constraint, a gotcha that cost time) → **first grep the index for an existing fact covering it**; update that file if found, else create a new one + index line.
2. A fact changed → update the fact file (never a second file on the same subject).
3. A fact invalidated → delete the file AND its index line.

**Fact-file format (exact):**
- Filename = kebab-case slug, same as frontmatter `name`.
- Frontmatter: `name`, `description` (one line, carries the read/skip decision), `metadata.type`: `project` | `feedback` | `reference`.
- Body ≤ 20 lines, ends with a `**How to apply:**` line; cross-link related facts with `[[name]]`.

**CAP:** index ≤ 25 fact lines. When exceeded: merge overlapping facts (types `project`/`feedback` first), delete anything failing the qualification test today, same commit. Memory is a working set, not an archive — history lives in git.
<!-- proto:end memory-protocol -->

<!-- proto:begin mirrors-lifecycle@1.4.0 -->
## Mirrors protocol (artifacts living in external systems)

`proto/mirrors/` holds the latest known copies of artifacts whose live version exists in an external system this repo cannot see or deploy to (cloud routine prompts, hosted agent configs, third-party workflow definitions).

The directory is created on demand: when the project's first such artifact appears, create `proto/mirrors/` with the artifact's file plus a `register.md` table (Mirror file | Live system | Deployed by | Verified) and keep the table current.

Lifecycle of a change — follow exactly:
1. Edit the mirror file in `proto/mirrors/` (and any affected `proto/context/` summary).
2. Commit + WORKLOG entry with **Open state: "edited, NOT deployed to live system yet"**.
3. A human deploys it to the live system (paste, upload, configure).
4. The change is verified live (dry-run or first real run observed).
5. WORKLOG entry closing the open state.

Never claim a mirrored change is "deployed" — only "edited" until steps 3–4 are confirmed. If live behaviour contradicts a mirror file, the live version has drifted: get the current live version and re-sync the mirror before editing anything.
<!-- proto:end mirrors-lifecycle -->

<!-- proto:begin connections-protocol@1.8.0 -->
## Connections protocol (proto/connections.md)

Registry of the systems reachable from this workspace — task trackers, knowledge bases, APIs, servers.

- **Mechanism** column values: `mcp` (MCP server) · `script` (code hitting an API) · `cli` (authenticated CLI) · `export` (dump pipeline) · `not yet connected`.
- **Verified** = the date the connection was last confirmed working. Stamp it whenever you rely on a row; a row without a recent date is a claim, not a fact.
- Record credential *locations*, never values.
<!-- proto:end connections-protocol -->

<!-- proto:begin session-close-checklist@1.7.0 -->
## Before ending a session (checklist)

1. WORKLOG entry appended (with a **Focus:** line if the session didn't advance the Milestone)? Required if anything beyond reading happened.
2. `proto/STATE.md` still true? If the position moved → rewrite it (Next, Milestone checkboxes, Standing, Waiting on, As-of date).
3. Record kept? New decisions → `DEC-###` entries; results now known → "Looking back" filled; out-of-scope ideas → `proto/IDEAS.md`; durable facts → `proto/memory/` (qualification test).
4. Externals flagged? Any `proto/mirrors/` edit pending deployment named in Open state; any stale `proto/context/` snapshot refreshed or flagged.
5. Everything committed? No orphan changes in the tree.
<!-- proto:end session-close-checklist -->

<!-- proto:begin change-discipline@1.8.0 -->
## The map: what each file holds and how it changes

Everything below sits inside `proto/` — the project's operational record, separated from the codebase at the root. Only CLAUDE.md lives at the root.

**Ownership rule:** the proto blocks in this file (`<!-- proto:begin … -->` … `<!-- proto:end … -->`) are framework-owned — never edit inside the markers; proto-update replaces block interiors wholesale. Project customizations live outside the markers, and on any conflict the text outside the markers wins.

| Path | Holds | How it changes |
|---|---|---|
| `CLAUDE.md` | Rules of operation only — including how to build/test/run the project | All other project knowledge belongs in `proto/`, never here |
| `proto/STATE.md` | One-screen position: Next / Goal / Roadmap / Milestone / Standing / Waiting on | Rewrite in place. Never grows, never archives |
| `proto/IDEAS.md` | Parked out-of-scope ideas | Curate: park cheaply, delete on promote/reject. Swept at milestone completion |
| `proto/WORKLOG.md` | Session-handoff journal | Newest-first. Past entries immutable — correct via a new entry. Rotates into monthly archives at the 500-line cap |
| `proto/memory/` | Durable repo facts + `MEMORY.md` index | Curate: update, merge, or delete fact files. Index capped at 25 lines |
| `proto/decisions.md` | Numbered decision record (`DEC-###`) | Newest-first, like the worklog. Later edits only **Status** and **Looking back**. Numbers never recycled. Rotates like the worklog |
| `proto/connections.md` | Registry of systems reachable from this workspace | Edit rows in place; stamp the Verified date whenever you confirm one |
| `proto/context/` | Project state snapshots (stamped with date + source) | Replace wholesale; re-stamp date + source |
| `proto/mirrors/` | Copies of externally-living artifacts + `register.md` | Created with the first mirror; sync per the mirrors lifecycle |
| `proto/docs/` | Long-form working documents (if the module is installed) | Add per the working-docs rules; mark superseded, never delete |
| `proto/archives/` | Rotated worklog and decision entries | Write-once; only rotation adds files here |
| `proto/VERSION` | The ProtoFramework version this workspace was initialized with | Only proto-init / proto-update touch it |
<!-- proto:end change-discipline -->

## Convention pointers

Shared conventions (task rules, team maps, org context) live in external systems — record pointers to them here, never copies of their content. Project data, not protocol — proto-update never touches this section.

{{CONVENTION_POINTERS}}

## How to work with the owner

- The **owner** is the human running this project — wherever a protocol says "the owner decides", it means them.
- Direct, concise, no fluff. Lead with what needs action.
- When a decision is made, log it in `proto/decisions.md`.
