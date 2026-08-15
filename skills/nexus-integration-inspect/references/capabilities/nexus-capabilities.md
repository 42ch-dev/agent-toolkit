# Nexus 能力基线（Capability Baseline）

| 元数据 | 值 |
|--------|----|
| 产品 | Nexus（本地优先、AI 驱动的叙事编排引擎；Rust workspace + TypeScript/Tauri 表面） |
| 调研日期（survey_date / research_date） | 2026-08-15 |
| surveyed HEAD SHA（full） | `7b04deafd64160eeead2972390b48189e21379f7` |
| surveyed HEAD SHA（short） | `7b04deaf` |
| HEAD 提交 | `Update Nexus description to include AI-driven feature` |
| 调研范围 | 只读；`../nexus`（crates/apps/packages/modules/docs/STRATEGY.md/CONCEPTS.md），README*/docs/STRATEGY/CONCEPTS、manifest 版本号、公开 API 面（CLI/HTTP/库/agent 接口）；未做逐 crate 源码盘点 |

**版本来源优先级**（对齐 technical-contract.md §2）：`Cargo.toml [workspace.package] version`（workspace manifest，本仓库为 `0.1.0`）> CHANGELOG（**`../nexus` 根部无 CHANGELOG.md，属预期回退路径**）> README/docs 声明版本；独立版本化发布包（npm/crates.io）以自身 manifest 为准（显式补充，见 README §版本跟踪约定）。所有 crates 经 `version.workspace = true` 继承 `0.1.0`；npm 包独立版本见各行 `version_source`。

## 能力表

| id | product | name | description | integration surface | version | version_source | survey_date | research_date | source_path |
|----|---------|------|-------------|---------------------|---------|----------------|-------------|---------------|-------------|
| nexus.cli.core | Nexus | nexus42 CLI | `nexus42` 单二进制同时承载 CLI 与 daemon（`daemon start`、`creator`、`system`、`platform`/`sync`、`acp`、`desktop`、`host-call` 等子命令），覆盖 World/Work/Manuscript/Knowledge 生命周期管理 | CLI（clap，Rust 二进制） | 0.1.0 | `../nexus/Cargo.toml [workspace.package] version` | 2026-08-15 | 2026-08-15 | ../nexus/README.md §Monorepo layout（另见 STRATEGY.md §What we build、apps/nexus42/src/cli.rs） |
| nexus.cli.connect | Nexus | nexus42 connect | `connect` 子命令族（`start` / `dial` / `peers list` / `token issue`）管理与 spoke-connect 对端：出站拨号、对端清单、能力令牌签发；仅在 `connect-host` feature 下编译 | CLI（feature-gated） | 0.1.0 | `../nexus/Cargo.toml [workspace.package] version`（connect-host feature；依赖 spoke-connect `=0.10.0`） | 2026-08-15 | 2026-08-15 | ../nexus/docs/nexus-runtime.md §Peer visibility / §Capability tokens（另见 apps/nexus42/src/cli.rs） |
| nexus.http-api.daemon | Nexus | Daemon API | daemon 进程通过 Axum 提供 `/v1/daemon/*` HTTP 面：worlds/kb/timeline/works/workspace/creators/findings/schedule/check/compute/inspector/memory（SOUL）/reading/preset-management/orchestration（sessions/capabilities）/agent-host scan/canvas；默认 loopback，远程绑定需 API key + TLS | HTTP（Axum，localhost / 可选远程 HTTPS） | 0.1.0 | `../nexus/Cargo.toml [workspace.package] version`（wire 类型另随 `@42ch/nexus-contracts` 0.30.0 版本化） | 2026-08-15 | 2026-08-15 | ../nexus/schemas/daemon-api/（另见 CONCEPTS.md §Daemon API、schemas/README.md §Product lines） |
| nexus.agent-api.acp | Nexus | ACP client | Nexus 是 ACP（Agent Communication Protocol）客户端而非服务端：通过 Agent Host 适配层请求用户本地 ACP agent 执行创作任务（`acp` 子命令 + daemon agent-host 面） | Agent 接口（ACP client / 本地 agent 调用） | 0.1.0 | `../nexus/Cargo.toml [workspace.package] version` | 2026-08-15 | 2026-08-15 | ../nexus/CONCEPTS.md §ACP / §Agent Host（另见 STRATEGY.md §Guiding Principles） |
| nexus.agent-api.connect-host | Nexus | Connect Host | 面向第三方 reasoner 的 opt-in 集成面：独立进程 `nexus-runtime`（headless）或 `nexus42 connect start`，经 spoke-connect 服务恰好六个 invoke op——`upsert` / `promote` / `relate` / `check` / `assemble` / `compute`；fail-closed allowlist + world/module 作用域 + capability token；`project`/未知 op 拒绝 | Agent 接口（spoke-connect 跨进程 invoke；libp2p 传输） | 0.1.0 | `../nexus/Cargo.toml [workspace.package] version`（connect-host feature；spoke-connect 依赖 `=0.10.0`） | 2026-08-15 | 2026-08-15 | ../nexus/docs/nexus-runtime.md（另见 CONCEPTS.md §Connect Host） |
| nexus.sdk.nexus-contracts | Nexus | @42ch/nexus-contracts npm package | 由 `schemas/` JSON Schema SSOT 生成的 TypeScript wire 类型（CJS/ESM + d.ts），Daemon API 与平台面的契约类型 | SDK（npm，TypeScript） | 0.30.0 | `../nexus/packages/nexus-contracts/package.json version` | 2026-08-15 | 2026-08-15 | ../nexus/packages/nexus-contracts/package.json（另见 README.md §Schemas & codegen） |
| nexus.sdk.spoke-adapter | Nexus | nexus-spoke-adapter Rust crate | SPOKE 集成边界 crate：Surface A（纯委托，冻结）+ Surface B（port trait + orchestrate 注入编排）双面 API；`extensions.nexus` 类型化命名空间访问器；pack 构建/解析；生产 `NexusAdapter` 承载基线 port 族 | 库（Rust crate，crates.io 发布面；依赖 spoke-schemas/spoke-operations `=0.10.0`） | 0.1.0 | `../nexus/Cargo.toml [workspace.package] version`（workspace 成员；消费 spoke 0.10.0） | 2026-08-15 | 2026-08-15 | ../nexus/crates/nexus-spoke-adapter/（另见 CONCEPTS.md §Extensions） |
| nexus.sdk.compute-module-abi | Nexus | WASM compute module ABI | 计算模块编写接口：`wasm32-unknown-unknown` 模块导出 `memory`/`alloc`/`compute`（可配置 `compute_export`，可选 `init`），宿主白名单 import（`nexus::kb_read` / `nexus::narrative_query`），`manifest.json`（含 `wasm_sha256`、`module_scope`），宿主本地安装、只读计算 | SDK/ABI（WebAssembly 模块契约，wasmtime 宿主） | 0.1.0 | `../nexus/Cargo.toml [workspace.package] version`（ABI 规范无独立版本号，fallback 至 workspace manifest） | 2026-08-15 | 2026-08-15 | ../nexus/docs/module-authoring.md（参考实现：modules/basic-combat/） |
| nexus.config.strategy-bundle | Nexus | External strategy bundle format | 策略包 = 纯目录（`preset.yaml` + `templates/`），声明 capability 路由与 prompt 模板（lanes/states/inner_graphs/run payload），经 validator 校验（`preset.id` == 目录名）；integrator 路径由第三方后端跑 LLM 并经 Connect 写回，daemon 路径由调度 API 驱动 | 配置格式（外部目录 bundle；validator + 调度消费） | 0.1.0 | `../nexus/Cargo.toml [workspace.package] version`（bundle 内 `preset.version` 为逐包 schema 版本，基线取 workspace fallback） | 2026-08-15 | 2026-08-15 | ../nexus/docs/strategy-authoring.md（worked examples：strategy-samples/） |
| nexus.data.knowledge-pack | Nexus | Knowledge Pack portable transfer format | 单个 JSON handbook 信封承载有序 KnowledgeEntry + Relation + SourceAnchor + `modules.pack` 目录元数据；CLI `creator world kb pack export/import` 与 daemon 路由 `POST /v1/daemon/worlds/:world_id/kb/pack/{export,import}`；三种冲突策略 skip/rename/overwrite；导入行带 `pack_import` 来源 | CLI + HTTP（产品自有传输格式，consumer-only） | 0.1.0 | `../nexus/Cargo.toml [workspace.package] version` | 2026-08-15 | 2026-08-15 | ../nexus/CONCEPTS.md §Knowledge Pack（另见 schemas/daemon-api/kb/pack-*.schema.json） |

## 说明

- `survey_date` 与 `research_date` 同义（brief 字段名 = contract `survey_date` 的别名），本文件两列同值 `2026-08-15`。
- 版本一致性：workspace manifest 版本 `0.1.0` 覆盖所有 `version.workspace = true` 的 crates 与 `nexus42`/`nexus-runtime` 二进制；npm 包（`@42ch/nexus-contracts` 0.30.0）独立版本化，其变更随 `wire_contracts_changed` 迭代节奏推进。
- 无 root CHANGELOG.md 属预期（contract 认可的回退路径），版本一律以 manifest 为准。
- 刷新与漂移检测：见同目录 README.md §刷新指引。
