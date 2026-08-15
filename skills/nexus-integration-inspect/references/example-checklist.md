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
| nexus.agent-api.connect-host | Connect Host | integrated | `src/connectClient.ts` 经 `@42ch/spoke-connect` 链接 nexus-runtime，用 capability token 调用 `compute` op（部署 allowlist 已授权） | 未使用其余五个 invoke op（非阻塞） | none |
| nexus.data.knowledge-pack | Knowledge Pack portable transfer format | integrated | `scripts/exportPack.ts` 调用 CLI `creator world kb pack export` 导出 handbook 备份 | — | none |
| nexus.http-api.daemon | Daemon API | not-applicable | 本项目为第三方运行时集成（nexus-runtime / spoke-connect 面）；daemon 是开发/调试态本地进程面（**非第三方运行时集成面**），未接入 | — | none |
| nexus.sdk.compute-module-abi | WASM compute module ABI | not-applicable | 项目为纯 TS 前端集成，无 wasm32 计算模块（未导出 memory / alloc / compute） | — | none |
| spoke.agent-api.operations | Adapter port + orchestration injection | not-integrated | 能力适用（项目需要 port 面与编排入口）但全仓 grep 无 `orchestrate*` 注入编排调用（仅手写 switch 按 ops 信封分发） | — | integration |

## 集成建议区（对应上表 not-integrated 行）

- **`spoke.agent-api.operations`**：使用现有能力接入。基线行给出接入面为库接口（TS + Rust parity）：经 `@42ch/spoke-connect/remote` 的 `connectRemoteAdapter`（自带调用方 `Transport`）建立 RemoteAdapter——port 调用以保留 `port.*` op 代理到已建立的 spoke-connect 会话（drop-in 异步 BaselinePorts 面），`orchestrateUpsert` / `orchestrateCheck` 等编排入口原样可跑；依赖版本 `@42ch/spoke-connect` 0.10.0（npm；Rust `spoke-connect` crate `=0.10.0` 的 `remote-adapter` feature 同语义，`connect_remote_adapter` → `Arc<RemoteAdapter>`）。参考位置：Spoke 仓库 `docs/how-to/connect-remote-adapter.md`（基线调研 HEAD `05915ad`）与 `packages/spoke-connect-ts/`（npm 包 `@42ch/spoke-connect`）。步骤：① 完成 ConnectHello 签名握手并授权对端 allowlist → ② `connectRemoteAdapter({ transport })` 建立 RemoteAdapter → ③ 用 `orchestrateUpsert` 验证 port 面（经 `port.*` 代理）→ ④ 按需扩展其余编排入口。

## 自研建议区（unlisted，独立章节）

- **需求**：项目需要多租户隔离的团队工作区管理（按租户切分 world 数据与权限），能力基线（`references/capabilities/nexus-capabilities.md` + `references/capabilities/spoke-capabilities.md` 全部行）中无匹配 capability id。
- **自研方向**：租户映射与权限在你方服务层实现——数据面仍走已集成的运行时集成面（`nexus.agent-api.connect-host` 经 Connect 写回）与 Knowledge Pack 传输格式（`nexus.data.knowledge-pack`）；不建议改动 Nexus/Spoke 侧。该需求可同步通过反馈流程（`nexus-feedback`）登记，供维护者评估是否纳入基线。
