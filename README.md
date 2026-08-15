# agent-toolkit

Nexus & Spoke's development toolkit for AI Agent — skills for third-party developers building agent integrations on [Nexus](https://github.com/42ch-dev/nexus) and [SPOKE](https://github.com/42ch-dev/spoke).

## Skills

| Skill | Purpose |
|-------|---------|
| `nexus-integration-inspect` | Check your project's integration status against the Nexus & Spoke capability baseline, item by item; get integration advice and self-development advice. |
| `nexus-feedback` | Turn session-context integration problems (issues, blockers, undeveloped needs) into a feedback report the Nexus & Spoke maintainers can act on. |

Each skill: `SKILL.md` (English, agent-facing trigger + workflow) + `references/` (Chinese, templates and rules).

## Install

```bash
npx skills add 42ch-dev/agent-toolkit
```

The `npx skills add` CLI copies the skills into your local skills directory (one subdirectory per skill under `skills/`); your agent picks them up via skill discovery. Point your coding agent at the skill folder or invoke by name (`nexus-integration-inspect`, `nexus-feedback`) as your agent tooling supports.

## Notes

- The capability baseline in `nexus-integration-inspect/references/capabilities/` is version-tracked (survey date + surveyed repo HEAD + per-capability version); refresh is maintainer-run when Nexus/Spoke publish new versions.
- Report outputs (`nexus-integration-checklist.md`, `nexus-feedback-report.md`) are written to your project/session workspace root — never into the skill directories.
