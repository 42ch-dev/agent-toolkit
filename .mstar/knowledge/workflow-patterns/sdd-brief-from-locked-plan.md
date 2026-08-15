---
module: mstar-harness
date: 2026-08-15
problem_type: workflow_issue
category: workflow-patterns
severity: medium
plan_id: 2026-08-nexus-integration-inspect
tags: [sdd, task-brief, plan-authoring, review-and-edit, dispatch]
---

# SDD task brief 必须在 Review & Edit chain 之后从最新 plan 生成

## Context

Morning Star 迭代 Phase 1 中，architect 会在 Review & Edit chain 里修订 plan（Task Files / acceptance / STOP 条件常常增删文件）。若 PM 在 chain 完成前按初稿写 SDD task brief，brief 与最终 plan 会不一致。

iter-2026-Q3 两 plan 各发生一次：brief 只列 4 个文件，plan（architect 修订后）要求 5 个（`example-checklist.md` / `example-report.md`），实现者按 brief 交付后被 task reviewer 抓到缺文件，额外补一轮 dispatch。

## Guidance

- **时机**：SDD task brief 在 plan lock（PM lock，compass `status: locked`）**之后**、首个 implement dispatch 之前生成；生成时重读磁盘上的 plan 文件（chain 已修订的版本），不要凭记忆或初稿。
- **验证**：brief 的 Files / Interfaces / Acceptance 与 plan Task 段落逐项核对；不一致时以 plan 为准并修正 brief。
- **防复发**：dispatch 前快速 diff brief 与 plan 的 Files 清单（grep 文件路径集合对比即可）。

## Why This Matters

brief 是 implementer 的唯一 spec（plan 不在 dispatch prompt 里）。brief 漏文件 = implementer 交付不完整 = 必然的 fix loop + 额外 dispatch 轮次；两 plan 都踩中，属于高复现过程缺陷。

## When to Apply

- 任何 `Execution mode: sdd` 的 plan，PM 写 task brief 时。
- plan 经过 specialist review-and-edit（product-manager/architect/writing-specialist）之后的首个 dispatch。

## Examples

- iter-2026-Q3 plan A：brief 4 文件 vs plan 5 文件（缺 `references/example-checklist.md`）→ 补建 + 补审。
- iter-2026-Q3 plan B：同样模式缺 `references/example-report.md` → 补建 + 补审。

## Source

- 经验来源：iteration:iter-2026-Q3（SDD 循环、task reviewer 发现、fix dispatch 记录）
- 相关产物：`.mstar/sdd/2026-08-nexus-integration-inspect/`、`.mstar/sdd/2026-08-nexus-feedback/`
