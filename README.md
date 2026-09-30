# ProtoFramework

**Give your AI agent a memory that survives the session.**

ProtoFramework is a set of protocols your coding agent follows so that long-running projects stay on track. It installs one `proto/` folder and one rules file into any repo. From then on, every session — today's, next week's, next month's — starts by knowing exactly where the project stands, what the current milestone is, and what to do first.

**What it does for you:**

- **Start a session with zero warm-up.** The agent reads one screen (`proto/STATE.md`) and knows the goal, the current milestone, what was done last time, and what to do first.
- **Finish what you started.** Every task is checked against the current milestone before work begins; good-but-off-topic ideas are parked in `IDEAS.md` and reviewed when the milestone completes — instead of derailing the week.
- **Ask "why did we do it this way?" months later and get an answer.** Decisions are numbered (`DEC-###`) with reasons, alternatives, and how they turned out; every session leaves a worklog entry you can grep.
- **Hand off between sessions, machines, or agents.** The record lives in the repo, in plain markdown, under git — not in one tool's chat history.
- **Straight, grounded answers.** Direct plain-language responses, under 50 words, each with a confidence score tied to how well it's backed by a named source. No assumptions — the agent asks before it guesses, and pushes back with reasons when a request would break something.
- **Works with your agent.** Claude Code, OpenCode, Goose, Codex — anything that reads `AGENTS.md` or `CLAUDE.md`.

**Use it on:** any project you work on with an AI agent over more than a couple of sessions — software, automations, research, content, ops. New repos or existing codebases.

---

## Getting started

**1. Install the skills** (once per machine):

```bash
npx skills add AGTGreg/ProtoFramework
```

**2. Open your project and initialize it.** In your agent session, say:

```
proto-init
```

Answer the short interview (project name, type, milestones, language). The skill scaffolds everything and commits it.

**3. Work normally.** The agent now orients itself at session start, logs as it goes, and closes sessions with the record up to date. Nothing extra for you to do.

**4. Update later.** When a new ProtoFramework version ships, say:

```
proto-update
```

Your protocol blocks get swapped to the latest version. **Your data is never touched.**

---

## What lands in your repo

```
your-project/
├── AGENTS.md          ← the protocol (all rules live here)
├── CLAUDE.md          ← 2-line shim: imports AGENTS.md for Claude Code
└── proto/
    ├── STATE.md       ← one screen: where the project stands, what's next
    ├── IDEAS.md       ← parking lot for out-of-scope ideas
    ├── WORKLOG.md     ← session journal, newest first
    ├── decisions.md   ← numbered decisions (DEC-###) with reasons & outcomes
    ├── connections.md ← external systems the project touches
    ├── memory/        ← durable project facts
    ├── context/       ← optional modules you pick at init
    │                     (architecture, environments, client, automations, docs)
    ├── archives/      ← rotated history
    └── VERSION        ← installed framework version
```

Your code owns the repo root. The framework owns one folder and two small files.

## How it works

- **One source of truth.** All protocol text lives in `AGENTS.md` inside versioned blocks (`<!-- proto:begin … -->`). OpenCode, Goose and Codex read it natively; Claude Code reads it through the `CLAUDE.md` import shim.
- **Blocks are framework-owned; everything else is yours.** `proto-update` swaps block interiors and never touches your data files, your custom sections, or your code.
- **Sessions are cheap.** Orientation is one bounded command (~1k tokens), not a pile of file reads. Deeper reads happen only when a task actually needs them.
- **The agent behaves.** Grounding rules (no assumptions, name your sources, push back with reasons) and plain-language rules (direct answers under 50 words, confidence %) are baked into the protocol — they apply no matter which agent or whose machine.

## Updating & versioning (maintainers)

- `PROTO_VERSION` — template semver. Every block carries its own `@version` in its marker.
- **MINOR/PATCH** = block-text or additive changes; `proto-update` applies them mechanically.
- **MAJOR** = anything that moves/renames/deletes files in target projects; the CHANGELOG entry carries a migration list that `proto-update` walks with the owner.
- Removed blocks get a tombstone in `manifest.json` → `removed_blocks`, so skip-version updates stay correct.
- Data files and module files are never updated in place — they're project data from the moment they're copied.

## Editing rules (contributors)

- Protocol changes go **inside** the marked blocks; bump the block `@version` and `PROTO_VERSION`; record in `CHANGELOG.md`.
- Placeholders are `{{UPPER_SNAKE}}` and must be listed in `manifest.json`. New files must be in the manifest (core, or a module with an `offer` line).
- Nothing organization-specific is ever baked into the template — specifics enter at init or during project work.
