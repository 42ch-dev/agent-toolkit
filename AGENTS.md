# AGENTS.md — agent-toolkit

Repository: `agent-toolkit` — Nexus & Spoke's development toolkit for AI Agent.

## Source Priority
1. Current user instruction
2. Root `AGENTS.md` (this file)
3. `.mstar/AGENTS.md` (Morning Star harness rules)
4. `mstar-*` skills

## Repository Boundaries
- Purpose: development toolkit for AI agents used by Nexus & Spoke.
- Currently scaffolding-level (LICENSE + README only); no build/test surface
  defined yet — do not invent one.

## Harness
- Morning Star harness active; harness root `.mstar/`.
- Process artifacts (plans, status.json, sdd/) are local and gitignored;
  results (`.mstar/AGENTS.md`, `knowledge/`, `specs/`) are tracked.
- Harness rules, path symbols, and gates: `.mstar/AGENTS.md`.

## Git
- Default branch: `main`. Feature work on dedicated branches per Assignment.
