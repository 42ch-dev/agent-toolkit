# skills/ — Agent Toolkit 技能索引

本目录存放 agent-toolkit 的技能，供第三方开发者在 Nexus & Spoke 上做 agent 集成时使用。每个技能由 `SKILL.md`（英文，触发契约与执行流程）+ `references/`（中文，模板与规则）组成。

| Skill | 用途 | 默认输出 |
|-------|------|---------|
| nexus-integration-inspect | 对照 Nexus & Spoke 能力基线逐项检查第三方项目的集成状态，输出检查清单 + 集成建议与自研建议 | `nexus-integration-checklist.md`（被检项目工作区根目录） |
| nexus-feedback | 从第三方开发者 session 上下文（+ 可选文件）提取 Nexus/Spoke 集成问题、blocker、未开发需求与采用缺口，输出维护者可直接排期的自包含反馈报告（平台待办 / adoption 分栏） | `nexus-feedback-report.md`（session 工作区根目录） |
| compute-module-author | 基于 V1.170 compute-module SDK 从零编写 Nexus compute 模块（scaffold → 实现 → build → validate → install → run），产出 `nexus42 compute validate` 通过的模块目录 | `dist/<module_id>/`（模块目录） |
