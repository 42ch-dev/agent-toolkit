# 检查清单模板 — nexus-integration-checklist.md

本模板是 `nexus-integration-inspect` 运行时输出 `nexus-integration-checklist.md` 的结构契约（technical-contract.md §3），同时被 `nexus-feedback` 只读复用。**保持列结构与状态闭集稳定**；变更需经 PM/architect 评审。

## 输出文件头

复制到输出文件顶部（漂移提示按实际情况填写）：

```markdown
# Nexus & Spoke 集成检查清单

- 被检项目：<项目名 / 路径>
- 检查日期：YYYY-MM-DD
- 能力基线：<surveyed HEAD SHA / 版本>（nexus-capabilities.md / spoke-capabilities.md 文件头）
- 版本漂移提示：<如有——被检项目使用版本 vs 基线调研版本差异；建议先跑刷新流程（技能 `references/capabilities/README.md` §刷新指引）>
```

## 空模板（复制到输出文件，逐行填写）

| capability id | 能力名 | 集成状态 | 证据 | 缺口 | advice_type |
|---|---|---|---|---|---|
|  |  |  |  |  |  |

- **capability id / 能力名**：从能力基线（`references/capabilities/nexus-capabilities.md` / `references/capabilities/spoke-capabilities.md`）**逐字复制**，不发明、不变换、不派生 id。
- **集成状态**：闭集 `integrated` | `not-integrated` | `not-applicable`（小写；仅此三值，不存在「部分集成」状态）。
- **证据**：每行必填——路径 / 引用 / 观察到的行为（能调用、有接入代码、配置了连接）。`not-integrated` 必须是**已核实的不存在**（verified absence）：证据不充分时，在本列记录搜索范围/局限（如「全仓 grep 仅覆盖 src/ 下 TS 文件」），或先问用户再定稿该行，不得默认判为 `not-integrated`。对 `not-applicable` 行，本列填写一行使用场景理由（该能力为何不适用于本项目）。
- **缺口**：可选——已集成能力上仍未被使用的表面（residual unused surface），**不是**第四种状态。
- **advice_type**：行级闭集 `integration` | `none`。`not-integrated` → `integration`（必填建议）；`integrated` / `not-applicable` → `none`。**自研建议不是行级 advice_type**，见下文「自研建议区」。

## 集成建议区

对每个 `not-integrated` 行写一段集成建议：如何使用**现有** Nexus/Spoke 能力补上（引用基线行的 integration surface、版本、示例位置）。规则与语气见 `references/advice-guide.md`。

## 自研建议区（独立章节）

被检项目需要、但能力基线**没有匹配 capability id** 的需求（unlisted）：逐条给出「需自行开发」的方向建议。**不得静默丢弃**；自研建议**不等于**、也不替代上面集成建议。判据与写法见 `references/advice-guide.md`。

## 状态判据

- **`integrated`**：为能力的预期用途实际使用——能调用（有接入代码 / 配置了连接 / 可观察到行为）；证据必填。该能力上未用的可选表面 → 写进「缺口」备注，不降级为 `not-integrated`。
- **`not-integrated`**：能力适用（符合项目用例），但未使用或不可用。必须给集成建议。
- **`not-applicable`**：明显超出本项目用例。**必须附一行使用场景理由**；不得标成 `not-integrated`。

## 示例行（trimmed row example，示意用假想项目「my-agent-demo」）

代表性示例（三种状态、集成建议区与自研建议区齐全，**精简子集**）见 `references/example-checklist.md`；下表为精简行示例：

| capability id | 能力名 | 集成状态 | 证据 | 缺口 | advice_type |
|---|---|---|---|---|---|
| spoke.sdk.typescript | @42ch/spoke-schemas + @42ch/spoke-operations (npm) | integrated | `package.json` 依赖 `@42ch/spoke-schemas@0.10.0`；`src/spokeClient.ts` 使用生成的 wire 类型构造 KnowledgeEntry / Relation | 未使用 spoke-operations 的生命周期助手（非阻塞） | none |
| nexus.agent-api.connect-host | Connect Host | not-integrated | 项目需外部 reasoner 触发 Nexus 编排，但全仓 grep 无 spoke-connect 客户端调用（无 `nexus42 connect` 拨号 / capability token） | — | integration |
| nexus.sdk.compute-module-abi | WASM compute module ABI | not-applicable | 项目为纯 TS 前端集成，无 wasm32 计算模块（未导出 memory / alloc / compute） | — | none |

### 示例：集成建议（对应上表 not-integrated 行）

- **`nexus.agent-api.connect-host`**：使用现有能力接入。基线行给出接入面为 spoke-connect 跨进程 invoke（恰好六个 op：`upsert` / `promote` / `relate` / `check` / `assemble` / `compute`），依赖版本 `spoke-connect =0.10.0`（crates.io；TS 参考客户端 `@42ch/spoke-connect` 同版本）。参考位置：Nexus 仓库 `docs/nexus-runtime.md`（peer visibility / capability tokens；基线调研 HEAD `7b04deaf`）与 Spoke 仓库 `packages/spoke-connect-ts/`（TS 参考客户端 npm 包 `@42ch/spoke-connect`；基线调研 HEAD `05915ad`）。步骤：① 接入 `@42ch/spoke-connect` 并完成 ConnectHello 签名握手 → ② 在部署配置 allowlist 中授权对端 → ③ 用签发的 capability token 调用六个 op 之一验证连通 → ④ 按需扩展其余 op。

### 示例：自研建议（unlisted，独立章节）

- **需求**：项目需要多租户隔离的团队工作区管理（按租户切分 world 数据与权限），能力基线中无匹配 capability id。
- **自研方向**：租户映射与权限在你方服务层实现——数据面可复用现有能力（daemon HTTP 面 `nexus.http-api.daemon`、Knowledge Pack 传输格式 `nexus.data.knowledge-pack`）按需接入；不建议改动 Nexus/Spoke 侧。该需求可同步通过反馈流程登记，供维护者评估是否纳入基线。
