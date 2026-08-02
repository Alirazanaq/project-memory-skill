---
name: project-memory
description: >
  Use this skill for any software project built or maintained with AI assistance — starting fresh, continuing an AI-built project, or inheriting an existing/legacy codebase with no AI-readable docs yet. Trigger on "vibe coding", "AI-built app", "debug my AI's code", "project memory", "context reset", "pick up where we left off", "why does this file exist", "scale this project", "audit this project", "scan this codebase", "what does this project actually do", "map this repo", "onboard me to this codebase", "find dead code", "check if these tests are real", or any request to set up documentation/change-tracking for a codebase. Self-detects whether the project is new, existing-undocumented, or already using this skill's docs, and adapts — no need to specify which. Do NOT trigger for trivial one-off scripts, or a narrow single-file question unrelated to a broader documentation/audit request.
---

# Project Memory & Debug Governance

## Role

You are not just writing code. You are acting as a **senior software engineer** responsible for a codebase that must stay understandable, debuggable, and handoff-ready — even after hundreds of AI-generated changes, even if your own context resets tomorrow, and even if you're the first AI ever to look at this project.

The core failure mode this skill prevents: an AI agent writes fast, working code with no record of *why*, so debugging or scaling later becomes guesswork — or inherits someone else's code with the exact same problem already baked in. Documentation is not overhead here — it is the only memory that survives a context reset, and the only ground truth when inheriting someone else's work.

## Core questions a project must always be able to answer

At any point, by reading the project's own files (not conversation history), you should be able to answer:

- What does this project do, and for whom?
- Why was this architecture / library / pattern chosen over alternatives?
- Which file owns which logic, and which files depend on which?
- When was a file created, changed, or deleted, and why?
- What is currently being worked on, and what's left?
- Is anything in this project fake — tests that pass without verifying behavior, dead files nothing uses?
- Could a different AI agent, with zero memory of this conversation, pick up exactly where you left off?

If the answer is "you'd have to scroll through chat history to know," or "you'd have to trust the README," the project has failed this skill's purpose.

## Team & Cross-Agent Use

This skill's biggest value shows up on a project touched by **more than one AI agent or person** — a team where members use different models and editors (Claude, Cursor, Copilot, Codex CLI, whatever), and where skill levels range from non-technical "vibe coders" to engineers who read every diff. In that setting, `/docs/` is not a personal convenience — it's the **shared source of truth** that makes everyone's tool-of-choice interchangeable and lets a new session (any agent, any platform) resume in seconds instead of re-scanning the whole codebase.

To actually deliver that, follow these on top of everything else in this skill:

- **Every session starts by reading `/docs/` first, regardless of which AI or editor is running it.** This is the whole point — it replaces re-scanning the codebase from scratch, which burns tokens and time every time someone switches chats or tools. Don't skip this because "I already know this codebase" — a fresh session never does.
- **Attribute every `LOG.txt` entry and every `TASKS.md` claim** — see the updated formats below. Anonymous entries defeat the purpose on a team; everyone needs to know who (which agent/model, or which person) did or is doing what.
- **Treat `/docs/` as version-controlled, like the code.** These files live in git alongside the source. Pull before starting work, commit/push the doc updates in the same commit as the code change they describe. A team desyncs the moment someone's docs update sits uncommitted on their machine.
- **DEBUG.md merge conflicts are resolved like any other tracked file** — normal git merge. If two sessions edited it simultaneously and the merge is unclear, `LOG.txt` (append-only, rarely conflicts) is the tiebreaker for what actually happened, since it's chronological and never overwritten.
- **Write DEBUG.md's summary so a non-technical teammate can read it.** A vibe coder shouldn't need to read a diff to know where the project stands — see the Plain-Language Status field in the DEBUG.md template. This is what removes the barrier between vibe coders and engineers on the same team: neither has to read the other's code to know what happened.

---

## Step 0 — Detect project state, every time you start

Before doing anything else, check `/docs/` at the project root:

- **`/docs/DEBUG.md` already exists** → this project already uses this skill. Skip to the normal Workflow section below.
- **`/docs/` doesn't exist, and there's little or no existing source code** → this is a **new project**. Go to Workflow → "Starting new work."
- **`/docs/` doesn't exist, but there's a real existing codebase** (inherited, legacy, AI-built without this skill, anything you didn't just create with the user) → run the **Audit Phase** below first. Do not start making changes to an undocumented existing codebase blind — establish ground truth first.

---

## Audit Phase (existing, undocumented codebases only)

This phase is **read-only** — it produces a report, it does not fix, refactor, or delete anything. Findings from here seed the documentation files in the next phase.

### A1 — Discover
- Get the directory tree, excluding `node_modules`, `.git`, `dist`, `build`, `vendor`, `__pycache__`, lockfiles, and other generated/dependency noise.
- Find the manifest (`package.json`, `pyproject.toml`, `requirements.txt`, `Cargo.toml`, etc.) and the entry point(s).
- Read the README if one exists — but treat it as a **claim to verify**, not a fact. A stale README is one of the most common findings.

### A2 — Infer the actual project goal
Synthesize from the manifest, README, entry point, routes/pages, and UI copy — not just the README's stated purpose. If the README claims one thing and the code clearly does another, report **both** and flag the mismatch explicitly rather than silently resolving it in either direction.

### A3 — File-by-file pass

For each source file: one-line purpose (inferred from **actual contents**, not filename or path), key functions/classes/exports, and a status (`active`, `entry point`, `dead`, `unclear`).

**"Scale to project size" means batching, not skimming.** On a large codebase it's tempting to eyeball folder names, read a handful of "important-looking" files, and generalize the rest — that produces a report that *looks* thorough but is guesswork wearing a table format. That's not acceptable. Every source file gets actually opened and read, even on a big project — the only thing that changes with size is how much depth each file gets and how the work is batched, never whether a file gets looked at.

### A3.5 — Full-coverage rules (read this before you start A3)

These rules exist specifically because large projects tempt an AI agent to go shallow. They are non-negotiable, not a suggestion:

1. **Never infer a file's purpose from its name or path alone.** `utils/helpers.js` is not "generic utility functions" until you've actually opened it and seen what's in it. A confident-sounding one-liner based on a guess is worse than an honest "unclear — see note."
2. **Process every file, not a sample.** For a large codebase, work through the file list in batches (e.g. 15-30 files at a time) rather than trying to read everything in one pass — but every batch continues until the full tree is covered. Don't stop at "I've seen enough to understand the pattern" and extrapolate the rest.
3. **Keep a coverage ledger while you work**, not just the final report table. Track: total source files found in A1, files actually opened and reviewed so far, files still pending. If you're tempted to wrap up the audit, check this ledger first — an incomplete ledger means the audit isn't done, regardless of how confident the summary feels.
4. **Depth may vary, presence may not.** A large, repetitive directory (e.g. 40 near-identical route handlers) can get one deep review of a representative file plus a fast pass confirming the others truly match that pattern — but "confirming they match" requires actually opening each one and checking, not assuming based on the first one and the folder name.
5. **Only truly non-logic files get the one-line pass without full reading**: pure config (e.g. `.eslintrc`, `tsconfig.json`), lockfiles, and already-excluded generated/vendor code from A1. Anything with actual application logic — including "boring" files like validators, constants, or small utility modules — gets read, because those are exactly the files a shallow pass skips and exactly where dead code and fake logic hide.
6. **State your actual coverage number in the report**, not a vague description. "Reviewed 187/187 source files (12 at full depth, 175 via pattern-confirmation against a reviewed representative)" is verifiable. "Reviewed the main files" is not and should never appear in the output.
7. **If you genuinely cannot complete full coverage in one pass** (context or time constraints), say so explicitly in the report, name exactly which files/directories weren't reached, and treat that as an unfinished audit rather than a finished one — don't let an incomplete pass quietly present itself as complete.

### A4 — Dependency mindmap
Trace imports/requires across files into a graph. Render as a Mermaid diagram — for large projects, group by directory/module rather than one node per file, and separately call out the 3-5 most-connected files (highest fan-in = riskiest to change).

### A5 — Dead and stale files
A **dead candidate**: nothing in the graph references it, and it isn't an entry point, config file, or externally-invoked script (check CI config and package.json scripts before concluding this — dynamic imports can hide real usage). A **stale** file: still referenced, but untouched for a long time (check `git log -1 --format=%ai -- <file>` if `.git` exists) while everything around it changed. Report both with reasoning and a confidence level. Don't delete anything.

### A6 — Fake / hollow test detection
Read `references/fake-test-patterns.md` before this step. For each test file, check what it actually asserts, not just that it passes. A 100%-green suite with near-zero real behavioral coverage is a finding worth flagging clearly — name the test, the pattern, and a concrete bug it would fail to catch.

### A7 — Gap analysis
Compare claimed functionality (README, manifest, UI copy, route names) against actual implementation. Look for stub functions (`pass`, `throw new Error("not implemented")`, `TODO`/`FIXME`, empty bodies), routes referenced on one side but missing on the other, broken imports, and inconsistent error handling. Report with file/line references, not vague impressions.

### A8 — Bootstrap the docs from findings
Instead of a separate standalone report, feed the audit directly into the Step 1 files below:
- `DEBUG.md` gets written from A2 (goal) and A3 (file map) — this is its first version, not an afterthought. Include the coverage number from A3.5 rule 6 in DEBUG.md's header so anyone reading it knows how thorough the baseline was.
- `TASKS.md` gets seeded with the gaps from A7 as tasks.
- `LOG.txt` gets one honest starting entry noting the state of the project as of this audit — dead files, fake tests, and gaps found — so the historical record doesn't start from a false "everything was fine" baseline. Use `Operation: Audit Baseline`.
- If fake tests or security-relevant gaps were found, note them in `SECURITY.md`'s Open Findings table too (see Security Review below).

Once this phase is done, proceed into the normal Workflow as if the project had used this skill from day one.

---

## The files

Create these under `/docs/` at the project root. Not every file is mandatory for every change — see the **Scope Rubric** below.

| File | Nature | Updated |
|---|---|---|
| `PRD.md` | Static-ish | Rarely, only on scope change |
| `TECHSTACK.md` | Static-ish | Rarely, only when a dependency changes |
| `ARCHITECTURE.md` | Semi-stable | On structural change |
| `DATABASE.md` | Semi-stable | On schema change |
| `API.md` | Semi-stable | On endpoint change |
| `TASKS.md` | Living | Continuously (checkboxes) |
| `DEBUG.md` | Living, **overwritten** | On every meaningful change |
| `SECURITY.md` | Config + living | Mode set once by user; findings appended by agent |
| `LOG.txt` (+ monthly archives) | **Append-only, never edited or deleted** | On every change |

Read `references/templates.md` for the exact structure of each file before creating them for the first time.

### The critical distinction: DEBUG.md vs LOG.txt

These two files must never duplicate each other's job, or they'll drift out of sync and someone will trust the wrong one:

- **DEBUG.md answers "what is true right now?"** A living document, overwritten in place — current architecture, file responsibilities, known issues, current TODOs. No history in it. When something changes, edit the relevant section; don't add a dated entry.
- **LOG.txt answers "what happened, and why, over time?"** Append-only, forever. Never edit or delete an old entry, even if the file it describes was later deleted — the entry stays as historical record.

If unsure which file something belongs in: "does this describe the current state, or an event that happened?" Current state → DEBUG.md. Event → LOG.txt.

### LOG.txt is not for wholesale reading

LOG.txt exists to be *searched*, not read start-to-finish — reading a large log defeats the purpose of this skill by burning the exact context budget it's meant to protect.

1. By default, do **not** read LOG.txt at all. DEBUG.md + TASKS.md should cover routine work.
2. Consult LOG.txt only when: (a) you need the history of a *specific* file or feature, or (b) a bug looks like it traces back to a past change.
3. When you do consult it, **grep/search for the specific filename, feature, or date range** — never load the whole file. If even a targeted search returns too much, roll old entries into a monthly archive (`logs/2026-07.txt`, current month stays as `LOG.txt`) — nothing is deleted, just moved out of the default read path.
4. LOG.txt's job is to capture **reasoning**, not diffs — if the project has git, it already stores line-by-line changes. Every entry should explain a *why* not visible in a diff. If there's no git, be more thorough about *what* changed too, since LOG.txt is then the only record.

Entry format:

```
===================================
<YYYY-MM-DD HH:MM>
Operation: Created | Modified | Deleted | Moved | Renamed | Security Fix | Security Flagged | Audit Baseline
Author: <agent/model name, e.g. "Claude Code", "Cursor+GPT", or the person's name if done manually>
File(s): <path(s)>
Reason: <why this change, what alternative was considered/rejected, if relevant>
Functions/Logic affected: <if applicable>
Severity: <Low | High — only for Security Fix / Security Flagged entries>
===================================
```

### TASKS.md is dynamic, not templated

Never impose a fixed task list. Read the PRD (or, for an audited existing project, the gap list from A7) and decide the breakdown yourself, sized to the actual project.

- Each task independently completable and testable before moving to the next.
- Mark progress inline: `[ ]` not started, `[~] <author>` in progress with who/which agent claimed it, `[x]` done.
- **On a team project, claim before you start, not after.** Before marking a task `[~]`, pull the latest `TASKS.md` and check nobody else already claimed it — two agents silently working the same task is exactly what this file exists to prevent.
- TASKS.md is also the single "what's active right now" source — no separate current-task file; keep the `[~]` item(s) current.

### Security Review

Security is reviewed and recorded with the same append-only discipline as everything else.

**Mode.** Check `/docs/SECURITY.md`. If it doesn't exist, ask the user which mode this project uses, then create it:
- **Auto-fix mode** — agent fixes security-critical issues directly and logs what it did.
- **Flag-only mode** — agent never applies a security-critical fix on its own; documents the finding and proposed fix, waits for user confirmation.

This is a per-project setting — ask once, then follow it without re-asking.

**When it triggers.** When a change falls into the Scope Rubric's "Full cycle" tier *and* touches auth, payments, user input handling, secrets/config, or a new dependency. Not on every change.

**How to review.** Prefer real tools (`semgrep`, `npm audit`, `pip-audit`, `bandit`, etc.) over LLM judgment alone where available; then do a manual pass for what tooling misses (hardcoded secrets, missing authorization checks, trusting client input).

**Severity decides what happens, regardless of mode:**
- **Low severity** — fix directly either way, log as a normal `Modified` entry.
- **High severity / security-critical** (auth bypass, exposed secrets, injection, broken access control, payments) — follow `SECURITY.md`'s declared mode.

Never downgrade severity to avoid asking — if unsure, treat as high. Every finding gets a `LOG.txt` entry, and if it reflects current posture, a note in `DEBUG.md`'s Known Issues until resolved.

---

## Scope Rubric — how much documentation a change needs

Don't apply the full doc cycle to every single change; that recreates the friction this skill removes. Default to the heavier tier when unsure.

**Full cycle required** (create/update PRD, TECHSTACK, ARCHITECTURE, DATABASE, API as relevant, plus DEBUG.md + LOG.txt) when:
- New project, or a new major feature area
- New external dependency/library introduced
- Database schema change
- Touches auth, payments, or security-sensitive logic — also triggers **Security Review** (above)
- Change spans 3+ files or introduces a new architectural pattern

**Lightweight** (just update TASKS.md checkbox + append one LOG.txt entry; DEBUG.md only if a file's responsibility changed) when:
- Single-function fix, copy/UI text change, 1 file, no new logic or dependency

This is a floor, not a ceiling — when genuinely unsure, do the full cycle; undocumented drift is worse than a few minutes of extra writing.

---

## Workflow

### Starting new work
1. Run Step 0 detection above first.
2. If `/docs/PRD.md` doesn't exist and this is genuinely a new project, write a **detailed** PRD and `ARCHITECTURE.md` first (get real input from the user on purpose, users, features, and structural decisions — don't guess to fill sections) — don't write implementation code before they exist. See `references/templates.md` for the full structure both should cover.
3. Read `TASKS.md` if it exists; otherwise generate it from the PRD (new project) or the audit gap list (existing project), sized to the actual project.
4. Pick the next task, mark it `[~]`.

### Before touching existing code
1. Read `DEBUG.md` for current state.
2. Only if the change is non-trivial or touches something you don't fully understand from DEBUG.md: search LOG.txt for that file/feature's history.
3. Never blindly overwrite logic you haven't understood first.

### After every change
1. Update the source code.
2. Apply the Scope Rubric to decide full vs lightweight documentation.
3. If the change qualifies (Security Review above), run it — fix/flag per `SECURITY.md`'s mode.
4. Update `DEBUG.md` in place if current state changed.
5. Append a `LOG.txt` entry (always — never skipped, even for lightweight changes).
6. Mark the task `[x]` in `TASKS.md`, or add new tasks if scope grew.
7. Quick self-check: does DEBUG.md actually match what the code now does?

### Debugging (never jump straight to editing code)
1. Read `DEBUG.md` — understand current implementation.
2. If the bug looks history-related, search `LOG.txt` for the relevant file/feature.
3. Locate affected files, find root cause.
4. Fix it.
5. Update `DEBUG.md` and append `LOG.txt` per the rules above.

### File-level headers
Every source file should open with: feature it belongs to, purpose, dependencies, created/last-modified date. Keep this local — don't duplicate it elsewhere.

---

## Golden rules

- Documentation earns its place by preventing a future guess, not by existing for its own sake.
- An undocumented existing codebase gets audited before it gets trusted — never assume an inherited README or a green test suite is telling the truth.
- On a large codebase, coverage doesn't shrink with size — depth per file can, but every file still gets opened and read. A confident-sounding summary built on guessed file purposes is a worse outcome than an honest, slower, complete pass.
- DEBUG.md is truth *now*. LOG.txt is truth *over time*. Never blend the two.
- LOG.txt entries are never edited or deleted — even for deleted files, even for reverted changes. Reversion gets its own new entry.
- Match documentation weight to the Scope Rubric — don't over-document trivial changes, don't under-document risky ones.
- Security-critical fixes follow whatever mode `SECURITY.md` declares — check it, don't assume it.
- Assume your own context can vanish at any moment. The project's `/docs/` folder is the only memory that survives that — for a new project or an inherited one.
