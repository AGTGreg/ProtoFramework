---
name: proto-update
description: Update a proto-initialized project to the latest ProtoFramework — swap outdated protocol blocks in CLAUDE.md, remove tombstoned blocks, never touch project data. Trigger on "proto-update", "proto update", "update the proto framework", "update the protocols", "bring the framework up to date".
---

# proto-update

Brings an initialized project's protocol blocks up to the template's versions. Touches ONLY the interiors of `<!-- proto:begin … -->` markers in CLAUDE.md, plus `proto/VERSION`. Everything else — data files, modules, text outside markers — is project property and is never modified.

## Template source

Same as proto-init: `PROTO_TEMPLATE_PATH` env var → local checkout; otherwise clone `https://github.com/AGTGreg/ProtoFramework` to a temp dir; otherwise ask for a path. Read the template's `PROTO_VERSION` and `manifest.json` (`blocks`, `removed_blocks`).

## Procedure (deterministic — follow in order)

### 1. Preflight

- Read the target's version stamp: `proto/VERSION`, or — legacy location from older installs — a root `PROTO_VERSION` file. Neither exists → ABORT: "Not proto-initialized — run proto-init instead." A root stamp is migrated at apply time (step 3).
- `git status --porcelain` dirty → STOP: ask the user to commit or stash first.
- Read target version T and template version L.
  - T == L → report "already up to date (vT)", stop.
  - T > L → STOP: "project is ahead of this template (T > L) — update your template checkout." Never downgrade.
  - **major(T) < major(L)** → STOP at the boundary: read the CHANGELOG entries for each MAJOR crossed, print their migration lists, and walk them with the owner step by step. Only after the migrations are done (and committed) continue with the block pass.

### 2. Plan (show before touching anything)

Build and print the plan, then ask the owner to confirm:

- **Swap:** every manifest block present in the target CLAUDE.md with an older `@version` than the template's (compare marker to marker).
- **Add:** every manifest block absent from the target — appended at the end of CLAUDE.md in template order, together with any non-block sections the blocks depend on that are missing ("Worklog tags", "Convention pointers", "How to work with the owner" — same list proto-init uses).
- **Remove:** every `removed_blocks` tombstone whose `removed_in` > T and whose block is still present — delete marker to marker; note the tombstone's disposition in the report.
- **Skip:** blocks already at the template version; blocks with a NEWER version than the template (warn — the project is partially ahead).

### 3. Apply

- Swaps/removals replace or delete **from begin marker to end marker inclusive** — never a character outside them.
- Placeholders inside newly added block text: fill from the project's existing content where evident (e.g. tags list already exists); otherwise leave a visible `<!-- TODO -->`.
- Write L into `proto/VERSION`. A legacy root `PROTO_VERSION` stamp → delete it (the stamp lives in `proto/` now).

### 4. Finish

- Commit exactly the files touched: `Update ProtoFramework protocols vT -> vL (<n> blocks swapped, <n> added, <n> removed)`.
- Print the report: swapped / added / removed / skipped / warnings, one line per block with old→new versions.
- **Flag, never touch (owner's manual cleanup):** legacy non-block sections whose content now lives in a block (e.g. an old "Where things live" section superseded by the map block), and old-style `proto/` data-file headers still carrying protocol text a block now owns (suggest slimming each to the one-line pointer). Both sit outside the markers — the ownership rule protects them, so the report only recommends.
- Remind: "Updated protocols apply from this point in the session onward."

## Guardrails (absolute)

- NEVER modify text outside proto markers — the ownership rule says outside text wins, and this skill is why.
- NEVER touch files under `proto/` other than `proto/VERSION`, and never module files — data files carry pointers, not protocol; there is nothing to update in them.
- NEVER downgrade a block or `proto/VERSION`.
- NEVER cross a MAJOR boundary without walking the CHANGELOG migration list with the owner first.
- Dirty tree → stop. No exceptions.
