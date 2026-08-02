# Document Templates

Read this file when creating one of these documents for the first time on a project. Don't load it for routine updates — you'll know the shape by then.

---

## PRD.md

This is a real product document, not a placeholder — fill every section with actual project-specific content, not generic filler. Get real answers from the user for anything you can't infer confidently; don't invent user personas or success metrics just to fill the section.

```markdown
# <Project Name> — PRD

## 1. Overview
2-4 sentences: what this is, in plain language.

## 2. Problem Statement
What problem exists today, for whom, and why it's worth solving. What happens if this isn't built.

## 3. Target Users / Personas
Who uses this. For each distinct user type: who they are, what they're trying to accomplish, what they currently do instead (if anything).

## 4. Goals & Success Metrics
What "working" looks like, ideally measurable. Not vague ("improve UX") — concrete ("user completes signup in under 60 seconds").

## 5. Features
For each feature:
- **Name**
- **Description** — what it does
- **User story** — "As a <user type>, I want to <action>, so that <outcome>"
- **Acceptance criteria** — the specific, checkable conditions that mean this feature is done
- **Priority** — must-have / should-have / nice-to-have

## 6. Functional Requirements
What the system must do, at a level more precise than the feature list — inputs, outputs, business rules, edge cases that matter.

## 7. Non-Functional Requirements
Performance targets, expected scale (users/data volume), security requirements, accessibility, browser/device support, uptime expectations — only the ones actually relevant to this project; don't pad with boilerplate that doesn't apply.

## 8. Assumptions & Constraints
What's being assumed (e.g. "users have modern browsers," "single-region deployment is fine for launch"). Real constraints (budget, timeline, must-use-existing-auth-system, etc.).

## 9. Out of Scope / Future
What's explicitly deferred, and why — so nobody re-litigates a decision that was already made.

## 10. Open Questions
Anything genuinely unresolved that needs an answer before or during build. Don't silently resolve these yourself and hide that a decision was made without the user.
```

For an audited existing project without a real PRD, reconstruct this from the Audit Phase's inferred goal (A2) — mark it clearly as reconstructed, not original: `_Reconstructed from codebase audit, <date> — no original PRD existed._` Fill what you can from evidence (features section can come from what's actually implemented); leave Open Questions or unclear sections honestly marked `_Unknown — not evident from code, ask the team._` rather than inventing an answer.

---

## TECHSTACK.md

```markdown
# Tech Stack

| Layer | Choice | Why |
|---|---|---|
| Frontend | ... | ... |
| Backend | ... | ... |
| Database | ... | ... |
| Auth | ... | ... |
| Deployment | ... | ... |

Note *why* for each — the alternative considered and rejected, if there was a real decision made. Skip the row entirely if it's not relevant to this project.
```

---

## ARCHITECTURE.md

This should let someone understand the whole system's shape without reading the code — go into real detail, not a one-paragraph sketch. Include actual diagrams (Mermaid), not just prose, wherever a relationship or flow is being described.

```markdown
# Architecture

## 1. System Overview
A short narrative of the overall shape (e.g. "SPA frontend, REST API backend, Postgres, background job queue for X") plus a high-level Mermaid diagram of the major components and how they connect.

## 2. Folder Structure
Every top-level directory, with what lives there and why it's organized that way — not just a tree dump, an explanation.

```
src/
  api/        — route handlers, one file per resource
  services/   — business logic, framework-agnostic
  db/         — schema, migrations, query layer
  ...
```

## 3. Component Breakdown
For each major component/module/service:
- **Responsibility** — what it owns, in one sentence
- **Inputs / Outputs** — what it receives and produces
- **Depends on** — what else it needs to function
- **Depended on by** — what breaks if this changes (pull this from the dependency mindmap if the project was audited)

## 4. Data Flow
Step-by-step, for the main flows that matter (e.g. "user submits form → validation → API call → service layer → DB write → response"). A Mermaid sequence or flow diagram per major flow, not just prose.

## 5. State Management (if applicable)
Where state lives (client, server, both), how it's synchronized, what the source of truth is for each piece of state.

## 6. Auth Flow (if applicable)
Full flow — login, token issuance, refresh, session handling, what's stored where and for how long. A diagram, not just a description.

## 7. External Integrations
Every third-party service/API this depends on: what it's used for, what happens if it's unavailable, where credentials are configured.

## 8. Deployment Architecture
Environments (dev/staging/prod), how a change gets from commit to running in production, infrastructure it runs on.

## 9. Scalability & Performance Considerations
Known bottlenecks, what was designed with future scale in mind vs. what's a deliberate "good enough for now," caching strategy if any.

## 10. Key Design Decisions & Tradeoffs
For each non-obvious architectural choice: what was chosen, what alternative was considered, why this one won. This is the section future developers most need and most often don't get — don't skip it.
```

Update on structural change — this doesn't mean re-writing the whole file every time; it means keeping each section accurate as the system evolves. For an audited existing project, §2-4 can be drawn directly from the Audit Phase's file map (A3) and dependency mindmap (A4); §10 may need to stay honestly marked `_Reasoning not evident from code — original decision-maker not available to ask._` where the "why" genuinely can't be recovered.

---

## DATABASE.md

```markdown
# Database

## Tables / Collections
| Name | Purpose | Key fields |
|---|---|---|

## Relationships
## Indexes & Constraints
```

---

## API.md

```markdown
# API

## <Route>
- Method:
- Input:
- Output:
- Auth required:
- Status codes:
```

---

## TASKS.md

Generate this from the PRD (new project) or the Audit Phase's gap list (existing project) — don't reuse a template task list, size it to the actual project.

```markdown
# Tasks

- [x] Task 1 — one line, done (Claude Code)
- [~] Task 2 — one line, in progress (currently here — claimed by Cursor+GPT, 2026-08-01)
- [ ] Task 3 — one line, not started
```

On a team/multi-agent project, always include who claimed an in-progress task and when — check this before claiming a task yourself.

Keep tasks independently completable. If scope grows mid-work, add tasks rather than silently expanding the current one.

---

## DEBUG.md

This is overwritten in place, never appended to. No dates-as-history here — LOG.txt owns history.

```markdown
# Debug — Current State

_Last updated: <date>_
_Audit coverage (if bootstrapped from an existing codebase): <e.g. "187/187 source files reviewed — 12 full depth, 175 pattern-confirmed">_

## Plain-Language Status
2-4 sentences, no jargon — what works today, what's being built right now, what's blocked. Written so a non-technical teammate (a "vibe coder," a PM, a stakeholder) can read this and know where the project stands without reading any code.

## Current Features
## Architecture Summary
(one paragraph, or point to ARCHITECTURE.md if it's already accurate)

## File Responsibilities
| File | Owns |
|---|---|

## Key Logic Worth Knowing
Anything non-obvious a new agent/dev would need explained.

## Known Issues / Current TODOs
```

For an audited existing project, the first version of this file comes directly from the Audit Phase's file-by-file pass (A3) and goal inference (A2) — this is DEBUG.md's day-one baseline, not written from scratch later.

---

## SECURITY.md

Created once, at project start (or before the first security-relevant change). Ask the user for the mode — don't assume it.

```markdown
# Security

## Mode
<Auto-fix | Flag-only>
(Set once by the user. High-severity findings follow this mode. Low-severity findings are always fixed directly regardless of mode.)

## Standing Checklist
- Input validation on all user-facing fields
- No secrets/credentials committed to source
- Authorization checked on every sensitive route, not just authentication
- Dependencies free of known critical CVEs
- (add project-specific concerns here)

## Open Findings
| Date | Severity | File(s) | Issue | Status |
|---|---|---|---|---|

(Move a finding out of this table once resolved — the resolution itself still gets its own LOG.txt entry.)
```

For an audited existing project, seed the Open Findings table with anything security-relevant found during the Audit Phase (fake tests around auth, missing input validation, etc.).

---

## LOG.txt entry format

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

Append only. Never edit or remove a past entry, even if the file it refers to was later deleted — the deletion gets its own new entry instead.

**`Audit Baseline` entry** — written once, at the end of the Audit Phase, for an existing project adopting this skill for the first time. Summarizes what the audit found (dead files, fake tests, gaps) as a single honest starting point, so the log doesn't silently begin from a false "everything was fine" assumption:

```
===================================
<YYYY-MM-DD HH:MM>
Operation: Audit Baseline
Author: <agent/model name that ran the audit>
File(s): <project root>
Reason: Adopting project-memory on existing codebase. Audit found: N dead file candidates, N hollow test patterns, N implementation gaps. See DEBUG.md and TASKS.md for details.
===================================
```

When the current month's LOG.txt grows large enough that a targeted search returns too much noise, move older entries into `logs/<YYYY-MM>.txt` archive files. Nothing is deleted — just relocated out of the default read path. The current month stays as `LOG.txt`.

---

## Source file header (top of each code file)

```
Feature: <feature this belongs to>
Purpose: <what this file does>
Dependencies: <other files/modules it relies on>
Created: <date>
Last Modified: <date>
```
