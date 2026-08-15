# 示例反馈报告 — nexus-feedback-report.md（完整 worked example）

本文件是 `nexus-feedback` 的**完整** worked example：一个第三方项目（示意用假想项目 `my-agent-demo`，TS/Node agent）在一次集成会话后生成的 `nexus-feedback-report.md` 全文示例。结构与字段遵循 `report-template.md`（technical-contract.md §4 锁定语义）：

- 六章节顺序固定；**分栏由 category 派生**：`issue` / `blocker` / `undeveloped-need` → 平台待办（第 3、4 节）；`adoption-gap` → 采用缺口（第 5 节），不混栏。
- 第 3/4/5 节每行六列齐全：`category` / `severity` / `product` / `capability id 或 unlisted` / `evidence` / `suggested action`。
- 非 `unlisted` 的 id 均**逐字**取自能力基线（`nexus-capabilities.md` / `spoke-capabilities.md`），可 grep 复核；至少 1 条 `unlisted` 与至少 1 条 `adoption-gap`。
- 示例行为是**完整演示形态**（区别于 `report-template.md` 内嵌的精简子集）：运行时报告只收录实际发现条目。

以下示例可直接复制为运行时输出形态：

```markdown
# Nexus & Spoke 集成反馈报告

- 报告方 / 项目：my-agent-demo（示意用假想项目，TS/Node agent）
- 报告日期：2026-08-16
- 会话环境：agent session（示意）
- 能力基线：nexus `7b04deaf` / spoke `05915ad`（2026-08-15 调研；nexus-capabilities.md / spoke-capabilities.md 文件头）
- 基线漂移提示：无（示例按基线版本填写）

## 1. 背景

my-agent-demo 是一个基于 Nexus & Spoke 的叙事 agent 集成（TS/Node）。本次会话尝试接入 Connect Host 远程触发编排、消费 Spoke ops wire 类型并导出 Knowledge Pack 备份；会话中遇到握手失败、类型不匹配与文档不一致等问题，另发现若干已发布能力适用但未采用，汇总如下供维护者排期。

## 2. 已集成项确认

| capability id | 能力名 | 确认方式 |
|---|---|---|
| spoke.sdk.typescript | @42ch/spoke-schemas + @42ch/spoke-operations (npm) | `package.json` 依赖 `@42ch/spoke-schemas@0.10.0`；`src/spokeClient.ts` 用生成的 wire 类型构造 KnowledgeEntry / Relation |
| spoke.sdk.connect | Connect client (TS + Rust reference) | `package.json` 依赖 `@42ch/spoke-connect@0.10.0`；`src/connectClient.ts` 建立 spoke-connect 会话（ConnectHello 握手已完成，阻塞在 auth challenge——见第 3 节） |
| nexus.data.knowledge-pack | Knowledge Pack portable transfer format | `scripts/exportPack.ts` 调用 `POST /v1/daemon/worlds/:world_id/kb/pack/export` 成功导出 |

## 3. 问题与 blocker（平台待办）

| category | severity | product | capability id 或 unlisted | evidence | suggested action |
|---|---|---|---|---|---|
| blocker | high | both | nexus.agent-api.connect-host | 会话报错原文：`ConnectAuthChallenge failed: token signature verification failed`；按 `docs/nexus-runtime.md` 用 `nexus42 connect start` 启动后用 `nexus42 connect token issue` 重新签发仍复现 | 排查 token 签发（nexus.cli.connect）与对端验证（spoke-connect 握手）链路；修复或给出临时 workaround 并在文档记录已知问题 |
| issue | high | Nexus | nexus.sdk.nexus-contracts | `package.json` pin `@42ch/nexus-contracts@0.29.0`，daemon 响应字段 `schedule` 与生成类型不符（类型错误原文：`Property 'schedule' is missing in type ...`）；基线调研版本为 0.30.0 | 核对 0.29 → 0.30 的 breaking changes 并补充升级说明 / 兼容记录 |
| issue | medium | Spoke | spoke.schemas.ops | `check` op 返回的 `ErrorEnvelope` 运行时缺少 TS 类型声明的 `code` 字段（报错原文：`error.code is undefined`） | 核对 `schemas/ops/` SSOT 与 codegen 输出，修复类型或 schema |
| issue | low | Nexus | nexus.cli.connect | `nexus42 connect start --help`（0.1.0）输出无 `--dial-timeout` 参数，而 `docs/nexus-runtime.md` §Peer visibility 的参数表列出该参数（文档与实现不符） | 核对 `apps/nexus42/src/cli.rs` 参数表与 `docs/nexus-runtime.md` §Peer visibility，统一文档或补参数 |

## 4. 未开发需求（平台待办）

| category | severity | product | capability id 或 unlisted | evidence | suggested action |
|---|---|---|---|---|---|
| undeveloped-need | medium | Nexus | unlisted | 会话中需要批量/增量同步多个 Knowledge Pack；基线 `nexus.data.knowledge-pack` 仅有单包 export/import，无匹配 id | 评估是否纳入基线（新增批量同步能力），或给出组合方案 |

## 5. 采用缺口（adoption）

| category | severity | product | capability id 或 unlisted | evidence | suggested action |
|---|---|---|---|---|---|
| adoption-gap | medium | Nexus | nexus.sdk.compute-module-abi | 项目有 wasm 计算需求，但当前用自家 HTTP 服务计算，未导出 `memory` / `alloc` / `compute` 模块 | 评估 compute-module-abi 的可发现性与示例完备性（非修复工单） |
| adoption-gap | low | Spoke | spoke.sdk.native-bindings | 项目服务端为 C#（.NET），当前仅用 TS SDK 直连，未使用 NuGet 原生绑定 | 评估 C# 绑定通道的文档/示例覆盖，降低采用门槛（非修复工单） |
| adoption-gap | low | Spoke | spoke.agent-api.operations | `src/spokeClient.ts` 手写 switch 按 ops 信封分发（upsert / relate / check），未使用 `@42ch/spoke-operations@0.10.0` 提供的 `orchestrateUpsert` / `orchestrateCheck` 编排助手 | 评估 orchestrate* 助手的可发现性与文档示例覆盖（非修复工单） |
| adoption-gap | low | Nexus | nexus.config.strategy-bundle | 项目用自有配置（`config/prompts.yaml`）声明 prompt 模板与 capability 路由，未采用外部 strategy bundle 格式（`preset.yaml` + `templates/`） | 评估 strategy bundle 对第三方 reasoner 的可发现性与示例完备性（非修复工单） |

## 6. 建议优先级

1. **high** — `nexus.agent-api.connect-host`（blocker，both）：握手失败阻断集成，优先排查。
2. **high** — `nexus.sdk.nexus-contracts`（issue，Nexus）：版本漂移导致类型不匹配，影响面大。
3. **medium** — `spoke.schemas.ops`（issue，Spoke）：ErrorEnvelope 类型缺陷。
4. **medium** — `unlisted`（undeveloped-need，Nexus）：批量 Knowledge Pack 同步需求。
5. **medium** — `nexus.sdk.compute-module-abi`（adoption-gap，Nexus）：wasm 计算走替代方案。
6. **low** — `nexus.cli.connect`（issue，Nexus）：CLI 参数与文档不符。
7. **low** — `nexus.config.strategy-bundle`（adoption-gap，Nexus）：strategy bundle 格式未采用。
8. **low** — `spoke.sdk.native-bindings`（adoption-gap，Spoke）：C# 绑定未采用。
9. **low** — `spoke.agent-api.operations`（adoption-gap，Spoke）：未用 orchestrate 编排入口。
```

## 示例自检说明

- **分栏派生**：第 3 节含 `blocker` ×1 + `issue` ×3，第 4 节 `undeveloped-need` ×1 → 平台待办；第 5 节 `adoption-gap` ×4 → adoption；第 3/4 节无 `adoption-gap` 行，第 5 节无 `issue` / `blocker` / `undeveloped-need` 行，不混栏。
- **必填字段**：第 3/4/5 节每行六列齐全（category / severity / product / capability id 或 unlisted / evidence / suggested action）；`category` ∈ {issue, blocker, undeveloped-need, adoption-gap}（小写），`severity` ∈ {high, medium, low}（小写），`product` ∈ {Nexus, Spoke, both}（首字母大写），均为闭集 token。
- **join 不变量**：非 `unlisted` 行共 8 个 id（`nexus.agent-api.connect-host`、`nexus.sdk.nexus-contracts`、`spoke.schemas.ops`、`nexus.cli.connect`、`nexus.sdk.compute-module-abi`、`spoke.sdk.native-bindings`、`spoke.agent-api.operations`、`nexus.config.strategy-bundle`）均可对 `skills/nexus-integration-inspect/references/capabilities/{nexus,spoke}-capabilities.md` grep 到原文；第 2 节 `spoke.sdk.typescript`、`spoke.sdk.connect`、`nexus.data.knowledge-pack` 同属基线 id；`undeveloped-need` 行为字面量 `unlisted`；`adoption-gap` 行均引用真实 id。
- **unlisted 占比**：占比 = 第 3+4+5 节 `unlisted` 行数 ÷ 第 3+4+5 节总行数（不含第 2 节已集成项确认）——本例 1 条 / 共 9 条 ≈ 11% ≤ ~30%，无需基线漂移提示。
- **evidence 自包含**：每条含报错原文（含命令名 / 文件路径）或项目文件路径 + 位置，无「见上方 session」。
