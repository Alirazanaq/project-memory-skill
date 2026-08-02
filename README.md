# project-memory

A Claude Skill that gives a team building software with AI a **shared, model-agnostic project brain** — so it doesn't matter that one teammate uses Claude, another uses Cursor, another is a non-technical "vibe coder" who's never opened a diff. Everyone, and every AI session, reads the same `/docs/` folder and knows exactly where the project stands.

No server, no database, no npm install, no MCP server to run. Just markdown files that live in your repo, next to your code, tracked by git like everything else.

## The problem this actually solves

Teams building with AI hit a specific, expensive pattern:

- Five people, five different AI tools/models, one codebase — nobody's context stays consistent
- Switch chats, restart a session, or hand off to a different tool → the agent re-scans the whole project from scratch, burning tokens and time it didn't need to
- A vibe coder on the team can't read the engineer's diffs, and the engineer doesn't want to explain every change in a meeting — there's a real barrier between them
- Nobody can say with confidence who changed what, when, or why — across five different tools, that history lives in five different, disconnected places (or nowhere)

`/docs/` fixes this by being the one place everyone — human or AI, regardless of platform — checks first and updates after every change. It's cheap to read (a few files), cheap to adopt (it's just markdown in your git repo), and it doesn't care which model wrote which line.

## What it does

This skill **self-detects** which situation it's in and adapts — you don't have to tell it:

- **New project, no code yet** → writes a detailed PRD and architecture doc, generates a dynamic `TASKS.md`, then works task by task with a running `DEBUG.md` (current state) and append-only `LOG.txt` (history) from day one.
- **Existing codebase, no docs** → runs a read-only **audit phase** first: maps every file's purpose and functions, builds a dependency mindmap, flags dead/stale files, hunts for fake/hollow tests (assertions that can't fail, mocks that replace the exact function under test, swallowed exceptions), and checks the README's claims against what the code actually does. Findings become the *first* version of `DEBUG.md` and `TASKS.md` — not a separate report nobody reads.
  - **No skimming on large projects.** The audit tracks a coverage ledger and requires every source file to actually be opened and read (depth can vary — e.g. one deep review + pattern-confirmation across 40 near-identical files — but presence can't be skipped). The report states a real coverage number ("187/187 files reviewed"), not a vague "reviewed the main files."
- **Already using this skill** → just follows the normal workflow below.

## The core distinction

`DEBUG.md` and `LOG.txt` are kept strictly separate so neither silently drifts out of trust:

| | `DEBUG.md` | `LOG.txt` |
|---|---|---|
| Answers | "What's true right now?" | "What happened, and why?" |
| Updated | Overwritten in place | Appended, never edited |
| Contains | Current architecture, file ownership, known issues | Dated entries: created/modified/deleted, with reasoning |

`LOG.txt` is meant to be **searched**, not read start to finish — reading a large log defeats the purpose of this skill by burning the exact context budget it's meant to protect. Old entries roll into monthly archives instead of growing one unbounded file.

## Also included: security review

Changes touching auth, payments, secrets, or user input trigger a security pass — real tools (`semgrep`, `npm audit`, `pip-audit`, etc.) preferred over LLM judgment alone. Low-severity findings are fixed directly; high-severity ones follow a mode you set once, per project, in `SECURITY.md`: auto-fix-and-log, or flag-and-wait-for-your-confirmation.

## Built for teams, not just solo sessions

- **Every `LOG.txt` entry and `TASKS.md` claim is attributed** — which agent/model, or which person, did or is doing what. No anonymous changes.
- **Claim before you start.** Before marking a task in-progress, the agent checks `TASKS.md` for an existing claim, so two team members' AI sessions don't silently duplicate work.
- **`/docs/` is git-tracked like the code** — pulled, committed, and pushed alongside it, so it stays in sync across everyone's machine and tool of choice.
- **`DEBUG.md` opens with a plain-language status** — a few jargon-free sentences on where the project stands, written so a non-technical teammate doesn't need to read a diff to follow along. This is what closes the gap between a "vibe coder" and an engineer on the same team.
- **New session, any platform, reads `/docs/` first** — instead of re-scanning the whole codebase every time someone switches chats or tools, which is where most of the wasted tokens in a multi-tool team actually go.

## Scope Rubric

Not every change needs the full documentation cycle — that would recreate the exact friction this skill removes.

**Full cycle** (PRD/architecture/DB/API docs + DEBUG.md + LOG.txt) when: new project/feature area, new dependency, DB schema change, anything touching auth/payments/security, or a change spanning 3+ files.

**Lightweight** (checkbox + one LOG.txt entry) when: single-function fix, copy change, 1 file, no new logic.

When unsure, default to the heavier tier.

## Install

**Claude Code / Claude.ai:** download `project-memory.skill` from [Releases](../../releases) and use "Save skill", or drop the folder into your skills directory.

**Manually:**

```
project-memory/
├── SKILL.md
└── references/
    ├── templates.md
    └── fake-test-patterns.md
```

## Files it creates in your project

```
/docs/
  PRD.md          — what the project is, for whom (reconstructed from audit if inherited)
  TECHSTACK.md    — choices + why
  ARCHITECTURE.md — structure, data flow
  DATABASE.md     — schema, relationships
  API.md          — routes, contracts
  TASKS.md        — living checklist, sized per-project (seeded from audit gaps if inherited)
  DEBUG.md        — current state, overwritten in place
  SECURITY.md     — security mode + standing checklist + open findings
  LOG.txt         — append-only change history (starts with an Audit Baseline entry if inherited)
  logs/YYYY-MM.txt — archived older log entries
```

Not every file is created for every project — see the Scope Rubric.

## Related work

Not the first take on AI-agent project memory — worth knowing before you adopt: Cline's Memory Bank, Claude Code's built-in `MEMORY.md`, [`skill-memory-bank`](https://github.com/fockus/skill-memory-bank), [`agentmemory`](https://github.com/rohitg00/agentmemory), and the emerging [`AGENTS.md`](https://agents.md) standard. This skill's niche: zero dependencies, single markdown file, nothing to install or run, and — unlike most of the above — it also knows what to do when you point it at a codebase you didn't build with it.

## License

MIT
