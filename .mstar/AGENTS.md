# AGENTS.md — `.mstar/` (Morning Star harness)

Morning Star harness rules for this repo. Harness lifecycle, state machine,
and skill index SSOT: `mstar-harness-core` and topic `mstar-*` skills.

## Source Priority
1. Current user instruction
2. Root `AGENTS.md`
3. This file
4. `mstar-*` skills

## Path Symbols (SSOT)
| Symbol | Path |
|--------|------|
| `{HARNESS_DIR}` | `.mstar/` |
| `{PLAN_DIR}` | `.mstar/plans/` |
| `{SDD_DIR}` | `.mstar/sdd/<plan-id>/` |
| `{ITERATION_DIR}` | `.mstar/iterations/` |
| `{KNOWLEDGE_DIR}` | `.mstar/knowledge/` |
| `{SPECS_DIR}` | `.mstar/specs/` |

## Content Boundaries
- `docs/`: human docs (installation, contribution)
- `.mstar/specs/`: frozen specs / ADRs — tracked
- `.mstar/knowledge/`: implementation SSOT, reusable designs — tracked
- `.mstar/plans/`: main plans, durable gate summaries — local, gitignored
- `.mstar/iterations/`: iteration packages (compass + guides/specs) — local, gitignored
- `.mstar/sdd/<plan-id>/review/`: QC/QA raw reports — local, gitignored

## State Machine & Status
- `Todo → InProgress → InReview → Done | Blocked`; `Done` only by
  `project-manager` or `qa-engineer`.
- `status.json` is SSOT for plan state and residuals
  (`residual_findings[<plan_id>]`); `notes.json` for session notes.
- Open R# are registered in `status.json`; archived only on closure.

## Gates
- Prepare: `specify → clarify → plan (locked)` before implement dispatch.
- Execute: `tasks → implement` (SDD default) → plan QC tri-review → QA gate → Done.
- Runtime/behavior changes need a recorded `QA gate` decision.
- Report-to-status sync is a hard gate before the next dispatch round.

## Git Tracking (process vs results)
- Tracked: `.mstar/AGENTS.md`, `.mstar/knowledge/**`, `.mstar/specs/**`
- Ignored: `plans/`, `iterations/`, `sdd/`, `notes.json`, `status.json`, `archived/`

Principle: process stays local; results are shared with the team.
Cross-clone persistent handoff = `.mstar/AGENTS.md` + `knowledge/` + `specs/`.
Do not `git add` `status.json` / `plans/` by default.
