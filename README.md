<div align="center">

# 🤖 agent-toolkit

**Nexus & Spoke's development toolkit for AI Agent**

Skills for third-party developers building agent integrations on [Nexus](https://github.com/42ch-dev/nexus) · [SPOKE](https://github.com/42ch-dev/spoke)

![License: Apache-2.0](https://img.shields.io/badge/license-Apache--2.0-blue.svg)
![Skills](https://img.shields.io/badge/skills-3-green.svg)

</div>

---

## ✨ Features

- **Check your integration** — item-by-item checklist against the published Nexus & Spoke capability baseline (`integrated` / `not-integrated` / `not-applicable`), with integration advice and self-development advice.
- **Report back to us** — turn session-context issues, blockers, and undeveloped needs into a structured feedback report the Nexus & Spoke maintainers can act on directly.
- **Author compute modules** — scaffold, implement, build, validate, install, and run a Nexus compute module on the V1.170 SDK, from zero to a `nexus42 compute validate`-green module directory.
- **Version-tracked baseline** — every capability carries its surveyed version, source, and date; refresh is maintainer-run when new versions ship.

## 📦 Skills

| Skill | Purpose | Key output |
|-------|---------|------------|
| [nexus-integration-inspect](skills/nexus-integration-inspect/) | Check your project's integration status against the Nexus & Spoke capability baseline, item by item | `nexus-integration-checklist.md` |
| [nexus-feedback](skills/nexus-feedback/) | Collect session-context integration problems (issues, blockers, undeveloped needs) into a maintainer-actionable feedback report | `nexus-feedback-report.md` |
| [compute-module-author](skills/compute-module-author/) | Author a Nexus compute module on the V1.170 SDK — scaffold, implement, build, validate, install, run | module dir green under `nexus42 compute validate` |

## 🚀 Quick start

```bash
npx skills add 42ch-dev/agent-toolkit
```

This copies the skills into your local skills directory (one subdirectory per skill under `skills/`). Your coding agent picks them up via skill discovery — invoke by name (`nexus-integration-inspect`, `nexus-feedback`) as your agent tooling supports.

### Usage

**Inspect your integration**

> "Check which Nexus & Spoke capabilities my project has integrated" → produces `nexus-integration-checklist.md` at your project root: one row per capability, plus integration advice and what to build yourself.

**Send feedback**

> "Summarize the Nexus/Spoke integration problems in this session as a feedback report" → produces `nexus-feedback-report.md` at your workspace root, consumable by the maintainers without the original session.

**Author a compute module**

> "Scaffold and build a Nexus compute module that resolves combat between two characters" → produces a module directory (from `modules/_template/` + the V1.170 SDK) that `nexus42 compute validate` accepts.

## 📄 License

[Apache-2.0](LICENSE)
