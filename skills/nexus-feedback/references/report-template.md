# 反馈报告模板 — nexus-feedback-report.md

本模板是 `nexus-feedback` 运行时输出 `nexus-feedback-report.md` 的结构契约（technical-contract.md §4）。**保持分栏结构与字段闭集稳定**；变更需经 PM/architect 评审。

与检查清单的区别：`nexus-integration-checklist.md` 一行一个基线能力（全量 21 行）；**反馈报告只收录本次会话发现的问题 / 未开发需求 / 采用缺口条目**，不是每个基线能力一行。

## 输出文件头

复制到输出文件顶部（基线漂移提示按实际情况填写）：

```markdown
# Nexus & Spoke 集成反馈报告

- 报告方 / 项目：<第三方项目名 / 路径>
- 报告日期：YYYY-MM-DD
- 会话环境：<模型 / 宿主 / 会话时间范围（可选）；会话内文件输入（可选）>
- 能力基线：<surveyed HEAD SHA / 版本>（nexus-capabilities.md / spoke-capabilities.md 文件头）
- 基线漂移提示：<如有——发现能力基线可能过期（unlisted 占比 > ~30% 或已知新版本）时填写；建议先跑 inspect 刷新流程再依赖本报告>
```

## 章节与字段规则

报告包含六个章节，顺序固定。**分栏由 category 派生（结构性约束，不是审阅者判断）**：`issue` / `blocker` / `undeveloped-need` → 平台待办（第 3、4 节）；`adoption-gap` → 采用缺口（第 5 节）。平台待办与 adoption 内容**不混栏**。

### 1. 背景

一段话说明报告来源：哪个第三方项目 / 什么集成场景 / 为什么提交这份反馈。让维护者无需原 session 也能理解上下文。

### 2. 已集成项确认

记录会话中确认**已经集成**的能力（或工作区内已有 `nexus-integration-checklist.md` 时，只读复用其 `integrated` 行），帮助维护者判断问题范围——个别能力上的缺陷不代表整体未集成。仅在有确认证据时填写；无确认项可省略本节的表并注明。

| capability id | 能力名 | 确认方式 |
|---|---|---|
|  |  |  |

- **capability id / 能力名**：从能力基线**逐字复制**，不发明、不变换、不派生。
- **确认方式**：路径 / 依赖声明 / 调用点 / 观察到的行为（与检查清单 `integrated` 行证据同标准）。

### 3. 问题与 blocker（平台待办）

会话中发现的平台缺陷、行为不符预期、阻断集成的问题。每行必填六列：

| category | severity | product | capability id 或 unlisted | evidence | suggested action |
|---|---|---|---|---|---|
|  |  |  |  |  |  |

- **category**：闭集 `issue` | `blocker`（小写）。`blocker` = 阻断或停止集成工作的问题；其余缺陷 = `issue`。
- **severity**：闭集 `high` | `medium` | `low`（小写）——用户影响面，与 category 正交。
- **product**：闭集 `Nexus` | `Spoke` | `both`（首字母大写）。
- **capability id 或 unlisted**：基线 id **逐字复制**，或字面量 `unlisted`（禁止编造 id）。若问题与已发布能力相关，必须填该能力 id。
- **evidence**：自包含——原样引用的会话报错 / 工具输出片段（含位置），或文件路径（含位置）。**禁止「见上方 session」**。
- **suggested action**：一条可执行的维护者动作（修复方向 / 排查路径 / 文档修正），不是空话。

### 4. 未开发需求（平台待办）

第三方需要、但能力基线**没有匹配 id** 的能力需求。列结构与第 3 节相同；`category` 恒为 `undeveloped-need`，`capability id` 恒为 `unlisted`。

> **消歧（强制）**：若该需求其实已有已发布能力覆盖或相关，**不是** `undeveloped-need`——改记该 id 上的 `issue`（按第 3 节），或按情况记 `adoption-gap`。判据见 `references/extraction-guide.md`。

### 5. 采用缺口（adoption）

第三方**未使用**的已发布能力（有基线 id、能力适用但开发者没用）。列结构同第 3 节；`category` 恒为 `adoption-gap`。

> **约束（强制）**：`adoption-gap` 必须引用真实基线 id（`capability_id` ≠ `unlisted`）——adoption 意味着「已发布能力未被使用」；这不是修复工单，`suggested action` 写采用侧动作（文档/示例/可发现性），不写平台缺陷修复。

### 6. 建议优先级

给维护者的排期视图：按 `severity` 降序（high → medium → low），同 severity 内按 product 分组；每条一行 = 「`capability id 或 unlisted` — 一句话」并指向第 3/4/5 节对应条目。让维护者扫一眼即可决定先处理什么。

## 落盘规则

- 默认写到 **session 工作区根目录** `nexus-feedback-report.md`（用户可指定其他路径）；**禁止**写入技能目录（`skills/nexus-feedback/`、`skills/nexus-integration-inspect/`）或 harness 目录（`.mstar/` 等）。
- 目标文件已存在时先问用户：覆盖 / 写时间戳文件，**不得静默覆盖**。
- 无占位符；示例只用于示意（见下）。

## 自检（每次落盘必跑）

1. 每行六字段齐全；`category` / `severity` 只出现闭集 token；`product` ∈ {Nexus, Spoke, both}。
2. 所有 `capability_id` ∈ 基线 id ∪ {`unlisted`}（逐字比对，可 grep 复核）。
3. `adoption-gap` 行无 `unlisted`；`undeveloped-need` 行全部为 `unlisted`。
4. 平台待办（第 3/4 节）与 adoption（第 5 节）不混栏。
5. 全文无「见上方 session」；每条 evidence 自包含。
6. `unlisted` 条目占比 ≤ ~30%；超限 → 在文件头写基线漂移提示并告知用户先跑 inspect 刷新流程，**不放松 join 规则**。

---

## 示例报告（partial worked example，示意用假想项目「my-agent-demo」）

本示例演示六章节、全部必填字段、平台待办与 adoption 分栏、至少 1 条 `unlisted` 与至少 1 条 `adoption-gap` 的写法。**示例行为精简子集**——运行时报告只收录实际发现条目；id 均逐字取自基线（非 `unlisted` 行可对基线文件 grep 复核）。

```markdown
# Nexus & Spoke 集成反馈报告

- 报告方 / 项目：my-agent-demo（示意用假想项目，TS/Node agent）
- 报告日期：2026-08-16
- 会话环境：agent session（示意）
- 能力基线：nexus `7b04deaf` / spoke `05915ad`（2026-08-15 调研；nexus-capabilities.md / spoke-capabilities.md 文件头）
- 基线漂移提示：无（示例按基线版本填写）

## 1. 背景

my-agent-demo 是一个基于 Nexus & Spoke 的叙事 agent 集成（TS/Node）。本次会话尝试接入 Connect Host 远程触发编排、消费 Spoke ops wire 类型并导出 Knowledge Pack 备份；会话中遇到握手失败、类型不匹配与文档不一致等问题，汇总如下供维护者排期。

## 2. 已集成项确认

| capability id | 能力名 | 确认方式 |
|---|---|---|
| spoke.sdk.typescript | @42ch/spoke-schemas + @42ch/spoke-operations (npm) | `package.json` 依赖 `@42ch/spoke-schemas@0.10.0`；`src/spokeClient.ts` 用生成的 wire 类型构造 KnowledgeEntry / Relation |
| nexus.http-api.daemon | Daemon API | `src/daemonClient.ts` 经 `/v1/daemon/*`（loopback + API key）调用 worlds / kb 路由 |
| nexus.data.knowledge-pack | Knowledge Pack portable transfer format | `scripts/exportPack.ts` 调用 `POST /v1/daemon/worlds/:world_id/kb/pack/export` 成功导出 |

## 3. 问题与 blocker（平台待办）

| category | severity | product | capability id 或 unlisted | evidence | suggested action |
|---|---|---|---|---|---|
| blocker | high | both | nexus.agent-api.connect-host | 会话报错原文：`ConnectAuthChallenge failed: token signature verification failed`；按 `docs/nexus-runtime.md` 用 `nexus42 connect start` 启动后用 `nexus42 connect token issue` 重新签发仍复现 | 排查 token 签发（nexus.cli.connect）与对端验证（spoke-connect 握手）链路；修复或给出临时 workaround 并在文档记录已知问题 |
| issue | high | Nexus | nexus.sdk.nexus-contracts | `package.json` pin `@42ch/nexus-contracts@0.29.0`，daemon 响应字段 `schedule` 与生成类型不符（类型错误原文：`Property 'schedule' is missing in type ...`）；基线调研版本为 0.30.0 | 核对 0.29 → 0.30 的 breaking changes 并补充升级说明 / 兼容记录 |
| issue | medium | Spoke | spoke.schemas.ops | `check` op 返回的 `ErrorEnvelope` 运行时缺少 TS 类型声明的 `code` 字段（报错原文：`error.code is undefined`） | 核对 `schemas/ops/` SSOT 与 codegen 输出，修复类型或 schema |

## 4. 未开发需求（平台待办）

| category | severity | product | capability id 或 unlisted | evidence | suggested action |
|---|---|---|---|---|---|
| undeveloped-need | medium | Nexus | unlisted | 会话中需要批量/增量同步多个 Knowledge Pack；基线 `nexus.data.knowledge-pack` 仅有单包 export/import，无匹配 id | 评估是否纳入基线（新增批量同步能力），或给出组合方案 |

## 5. 采用缺口（adoption）

| category | severity | product | capability id 或 unlisted | evidence | suggested action |
|---|---|---|---|---|---|
| adoption-gap | medium | Nexus | nexus.sdk.compute-module-abi | 项目有 wasm 计算需求，但当前用自家 HTTP 服务计算，未导出 `memory` / `alloc` / `compute` 模块 | 评估 compute-module-abi 的可发现性与示例完备性（非修复工单） |
| adoption-gap | low | Spoke | spoke.sdk.native-bindings | 项目服务端为 C#（.NET），当前仅用 TS SDK 直连，未使用 NuGet 原生绑定 | 评估 C# 绑定通道的文档/示例覆盖，降低采用门槛（非修复工单） |

## 6. 建议优先级

1. **high** — `nexus.agent-api.connect-host`（blocker，both）：握手失败阻断集成，优先排查。
2. **high** — `nexus.sdk.nexus-contracts`（issue，Nexus）：版本漂移导致类型不匹配，影响面大。
3. **medium** — `spoke.schemas.ops`（issue，Spoke）：ErrorEnvelope 类型缺陷。
4. **medium** — `unlisted`（undeveloped-need，Nexus）：批量 Knowledge Pack 同步需求。
5. **medium** — `nexus.sdk.compute-module-abi`（adoption-gap，Nexus）。
6. **low** — `spoke.sdk.native-bindings`（adoption-gap，Spoke）。
```

> **示例自检说明**：上表非 `unlisted` 的 id（`nexus.agent-api.connect-host`、`nexus.sdk.nexus-contracts`、`spoke.schemas.ops`、`nexus.data.knowledge-pack`、`nexus.sdk.compute-module-abi`、`spoke.sdk.native-bindings`、`spoke.sdk.typescript`、`nexus.http-api.daemon`）均可对基线文件 grep 到原文；`unlisted` 1 条 / 共 6 条 ≈ 17% ≤ ~30%；`adoption-gap` 行均引用真实 id；`undeveloped-need` 行为 `unlisted`；平台待办与 adoption 分栏不混。
