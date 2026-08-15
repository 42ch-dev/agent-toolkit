---
module: agent-toolkit
date: 2026-08-15
problem_type: api_design
category: api-design
severity: medium
plan_id: 2026-08-nexus-integration-inspect
tags: [nexus, spoke, capability-baseline, skill-contract, capability-id]
---

# Nexus & Spoke 跨技能数据契约（capability id / baseline / checklist / feedback）

## Context

agent-toolkit 面向第三方开发者提供两个协作技能：`nexus-integration-inspect`（对照能力基线检查集成状态）与 `nexus-feedback`（从 session 提取集成反馈）。两技能共享同一份能力基线且按 capability id 做 join。若无机器可校验的契约，技能间会漂移（编造 id、状态词不一致、列语义分叉）。

## Guidance

- **Capability id**：`<product>.<area>.<slug>`，正则 `^(nexus|spoke)\.[a-z][a-z0-9-]*\.[a-z][a-z0-9-]*$`；product 必须等于行 `product` 字段；`<area>` 来自封闭词表（README 维护，永不改名）；id 一旦发布不可变（改名 = 新 id + 旧 id 标记 deprecated 指向后继）；跨基线文件全局唯一。
- **Baseline 行**：每条能力含 id / name / description / integration surface / version / version_source / survey_date（canonical；research_date 已废弃为别名）/ source_path；头部记 surveyed HEAD SHA + 版本来源优先级（workspace > CHANGELOG > README/docs）。
- **Checklist 状态**：闭集 `integrated | not-integrated | not-applicable`（无 partial；not-applicable 必须附一行使用场景理由）；行级 advice_type 闭集 `integration | none`；自研建议是独立章节（只收无 id 的需求），**不是**行级值。
- **Feedback 条目**：category 闭集 `issue | blocker | undeveloped-need | adoption-gap`；severity 闭集 `high | medium | low`（与 category 正交）；分栏**由 category 派生**（issue/blocker/undeveloped-need → 平台待办；adoption-gap → adoption）；id verbatim join，无匹配 → `unlisted`（禁止编造 id）。
- **判定词**：同一能力的行为缺陷 → 该 id 上的 `issue`；能力范围之外的新需求形态 → `undeveloped-need`/`unlisted`。
- **unlisted 占比 STOP**：占比 = 第 3+4+5 节 unlisted 行数 ÷ 第 3+4+5 节总行数（不含第 2 节已集成确认，其 id 天然非 unlisted）；> ~30% → STOP 上报（基线可能过期，刷新后重跑）。
- **跨技能文件契约**：基线单一落点（`nexus-integration-inspect/references/capabilities/`，只读复用）；运行时输出（`nexus-integration-checklist.md` / `nexus-feedback-report.md`）落在被检项目/session 工作区根，永不写入技能目录或 harness 路径；覆盖写前询问（时间戳文件名回退）；用户指定路径落入禁止目录时拒绝。

## Why This Matters

两技能独立演进时，id join 与状态/列语义是唯一的机器可校验锚点。契约锁死闭集与 verbatim 规则后，QC 可以用 grep 机械验证（正则、`sort | uniq -d`、闭集 token 扫描），无需运行时代价。

## When to Apply

- 在 agent-toolkit 内新增任何消费能力基线的技能。
- 基线刷新（Nexus/Spoke 新版本）时按 README §刷新指引逐行更新，id 不可变，仅新增/废弃。

## Examples

- 正确 id：`spoke.agent-api.operations`、`nexus.sdk.compute-module-abi`；错误：`spoke.Operations`、`nexus.observability.logs`（area 不在词表）、`spoke.sdk.typescript.v2`（4 段）。
- 反馈条目分类：Connect 六操作中某操作报错 → `issue` @ `nexus.agent-api.connect-host`；需要「批量增量同步」而基线只有单包 export/import → `undeveloped-need`/`unlisted`（evidence 注明相关能力）；项目没调用已发布的 CLI 命令 → `adoption-gap` @ `nexus.cli.connect`。

## Source

- 源规范：`.mstar/iterations/iter-2026-Q3/specs/technical-contract.md`（iteration:iter-2026-Q3，提升前路径）
- 实现：`skills/nexus-integration-inspect/`、`skills/nexus-feedback/`（iter-2026-Q3，QC tri 3/3 Approve ×2）
