# 示例检查清单 — 代表性示例（partial worked example，假想项目「my-agent-demo」）

本文件是运行时输出 `nexus-integration-checklist.md` 的**代表性示例（精简子集）**（假想项目「my-agent-demo」，示例行为 `references/checklist-template.md` §示例行的精简子集），供编写输出清单时对照，演示三种集成状态（`integrated` / `not-integrated` / `not-applicable`）、行级 `advice_type` 闭集（`integration` | `none`）、集成建议区与自研建议区的写法。列结构、状态判据与建议规则以 `references/checklist-template.md` 与 `references/advice-guide.md` 为准；本文件不是运行时输出，不替代被检项目的真实检查结果。

> **注意（运行时约束）**：示例行为**精简子集**——运行时输出 `nexus-integration-checklist.md` 必须包含**全部基线行（当前 21 行）**，一行一个能力；本文件仅演示状态与建议（集成建议区 / 自研建议区）的写法，不代表运行时输出可以只有这些行。

## 输出文件头（示例）

```markdown
# Nexus & Spoke 集成检查清单

- 被检项目：my-agent-demo（示意用假想项目）
- 检查日期：2026-08-16
- 能力基线：nexus `7b04deaf` / spoke `05915ad`（2026-08-15 调研；nexus-capabilities.md / spoke-capabilities.md 文件头）
- 版本漂移提示：无（示例按基线版本填写；实际存在差异时写明差异并建议先跑 `references/capabilities/README.md` §刷新指引）
```

## 检查清单（示例行）

| capability id | 能力名 | 集成状态 | 证据 | 缺口 | advice_type |
|---|---|---|---|---|---|
| spoke.sdk.typescript | @42ch/spoke-schemas + @42ch/spoke-operations (npm) | integrated | `package.json` 依赖 `@42ch/spoke-schemas@0.10.0`；`src/spokeClient.ts` 使用生成的 wire 类型构造 KnowledgeEntry / Relation | 未使用 spoke-operations 的生命周期助手（非阻塞） | none |
| nexus.http-api.daemon | Daemon API | integrated | `src/daemonClient.ts` 经 `/v1/daemon/*` HTTP 面（loopback + API key）调用 worlds / kb / findings 路由 | 未使用 schedule / preset-management 路由（非阻塞） | none |
| nexus.data.knowledge-pack | Knowledge Pack portable transfer format | integrated | `scripts/exportPack.ts` 调用 `POST /v1/daemon/worlds/:world_id/kb/pack/export` 导出 handbook 备份 | — | none |
| nexus.agent-api.connect-host | Connect Host | not-integrated | 项目需外部 reasoner 触发 Nexus 编排，但全仓 grep 无 spoke-connect 客户端调用（无 `nexus42 connect` 拨号 / capability token） | — | integration |
| nexus.sdk.compute-module-abi | WASM compute module ABI | not-applicable | 项目为纯 TS 前端集成，无 wasm32 计算模块（未导出 memory / alloc / compute） | — | none |

## 集成建议区（对应上表 not-integrated 行）

- **`nexus.agent-api.connect-host`**：使用现有能力接入。基线行给出接入面为 spoke-connect 跨进程 invoke（恰好六个 op：`upsert` / `promote` / `relate` / `check` / `assemble` / `compute`），依赖版本 `spoke-connect =0.10.0`（crates.io；TS 参考客户端 `@42ch/spoke-connect` 同版本）。参考位置：Nexus 仓库 `docs/nexus-runtime.md`（peer visibility / capability tokens；基线调研 HEAD `7b04deaf`）与 Spoke 仓库 `packages/spoke-connect-ts/`（TS 参考客户端 npm 包 `@42ch/spoke-connect`；基线调研 HEAD `05915ad`）。步骤：① 接入 `@42ch/spoke-connect` 并完成 ConnectHello 签名握手 → ② 在部署配置 allowlist 中授权对端 → ③ 用签发的 capability token 调用六个 op 之一验证连通（令牌签发走 `nexus42 connect token issue`，基线 `nexus.cli.connect`）→ ④ 按需扩展其余 op。

## 自研建议区（unlisted，独立章节）

- **需求**：项目需要多租户隔离的团队工作区管理（按租户切分 world 数据与权限），能力基线（`references/capabilities/nexus-capabilities.md` + `references/capabilities/spoke-capabilities.md` 全部行）中无匹配 capability id。
- **自研方向**：租户映射与权限在你方服务层实现——数据面仍走已集成的 daemon HTTP 面（`nexus.http-api.daemon`）与 Knowledge Pack 传输格式（`nexus.data.knowledge-pack`）；不建议改动 Nexus/Spoke 侧。该需求可同步通过反馈流程（`nexus-feedback`）登记，供维护者评估是否纳入基线。
