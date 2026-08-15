# capabilities/ — Nexus & Spoke 能力基线（共享调研产物）

本目录是 **nexus-integration-inspect** 与 **nexus-feedback** 共享的只读能力基线（跨技能 join 键 = capability `id`，id 契约见下方 §id 契约）。任何技能不得在此目录之外另建能力目录。

- `nexus-capabilities.md` — Nexus 能力基线（surveyed HEAD `7b04deaf`，2026-08-15）
- `spoke-capabilities.md` — Spoke 能力基线（surveyed HEAD `05915ad`，2026-08-15）

## 调研方法（有界调研）

**范围（只读，硬边界）**：
1. README / README_CN / `docs/` / STRATEGY.md / CONCEPTS.md / CHANGELOG.md（如有）
2. manifest 版本号：`Cargo.toml`（workspace 根 + 关键 crate）、`Package.swift`、`go.mod`、npm `package.json`
3. 公开 API 面：CLI 入口（clap 命令树）、HTTP 服务面（路由/JSON Schema 目录）、库/模块公开接口（crate/npm 包说明）、agent 可用的集成面（ACP / spoke-connect / 适配器 port）

**禁止**：
- 逐 crate 深度源码盘点（每条能力的证据必须是文档/manifest/公开接口面，不是实现细节）
- 修改被调研仓库（Nexus/Spoke 检出）任何文件
- 向任何仓库内部过程目录写入

**数据来源约定**：每条能力行含 `source_path`（证据来源路径）与 `version_source`（版本来源文件 + 字段），保证可溯源；无法溯源到版本号 + 路径的能力不落盘。`source_path` 为**相对被调研仓库检出根**的路径（如 `crates/nexus-spoke-adapter/`、`packages/spoke-schemas/`；产品列标识所属仓库），指向 crate/包目录时是**包定位符（package locator）**而非实现文件引用——证据是该目录内 manifest/README/公开接口面；逐 crate 读实现仍属越界（见上「禁止」）。

## `<area>` 封闭词表（id 第二段）

由 Task 1 调研产出，共 **7 个 area**；新增 area 必须先更新本表（经 PM/architect 评审），**已发布 area 永不改名**（area 是 id 的不可变部分）。

| area | 含义 | 示例行 |
|------|------|--------|
| `cli` | 命令行接口面（二进制/子命令） | nexus.cli.core、nexus.cli.connect |
| `http-api` | HTTP 服务面（daemon 本地进程面为开发/调试态，非第三方运行时集成面） | nexus.http-api.daemon |
| `agent-api` | agent/协议交互面（ACP、spoke-connect、适配器 port 编排） | nexus.agent-api.connect-host、spoke.agent-api.operations |
| `sdk` | 语言 SDK / 发布包 / ABI / 参考实现（npm、crates.io、FFI、WASM ABI） | nexus.sdk.nexus-contracts、spoke.sdk.native-bindings |
| `schemas` | JSON Schema wire 契约（SSOT 目录） | spoke.schemas.data、spoke.schemas.ops、spoke.schemas.connect |
| `data` | 传输/内容格式（产品自有数据形状、方言机制） | nexus.data.knowledge-pack、spoke.data.domain-profiles |
| `config` | 外部配置/清单格式（bundle、manifest 文件格式） | nexus.config.strategy-bundle |

**归类约定**：
- 同一能力面有多个承载（如 Connect 同时有 schemas/sdk/agent-api 三层）时，按"消费者接触的那一层"分 area，允许同一主题跨 area 多行（id 仍唯一）。
- **运行时集成面 vs 开发面**：`nexus.agent-api.connect-host`（nexus-runtime / spoke-connect 跨进程 invoke）是第三方运行时集成主面；`nexus.http-api.daemon` 为开发/调试态面（daemon 本地进程面），**不是第三方运行时集成面**。检查清单评估 daemon 行时按「开发态」对待：本地开发工具可 `integrated`，纯运行时集成为 `not-applicable`（附理由）或 `not-integrated`。
- 归类争议影响 > 约 10% 条目时 STOP 上报，不得现场发明新 area 或破坏三段式 id。

## id 契约（不可变，technical-contract.md §1）

- 形状：`<product>.<area>.<slug>`，三段小写点分；`product` ∈ {nexus, spoke} 且必须与该行 `product` 字段一致。
- 校验 regex：`^(nexus|spoke)\.[a-z][a-z0-9-]*\.[a-z][a-z0-9-]*$`。
- 唯一性：跨 `nexus-capabilities.md` + `spoke-capabilities.md` 两文件全局唯一。
- 不可变性：id 一经发布含义不变；改名/重定范围 → 发新 id，旧 id 在本 README 记为 deprecated 并指向后继者（本基线尚无 deprecated id）。
- join 规则：nexus-feedback 逐字复制基线 id，绝不发明/变换/派生 id；无匹配基线 id 的第三方需求记 `unlisted`。

## 版本跟踪约定

- 行字段：`version`（来源版本号）+ `version_source`（来源文件 + 字段）+ `survey_date`（**规范列**，= 本次调研日 2026-08-15）+ `source_path`（证据路径）。`research_date` 为 **deprecated 别名**（与 `survey_date` 同值，仅向后兼容）；**刷新时只写 `survey_date`，不再双写**；本次不改写既有行值。
- **版本来源优先级**（对齐 technical-contract.md §2）：workspace manifest 版本（`Cargo.toml [workspace.package] version`）> CHANGELOG 最新条目 > README/docs 声明版本；胜者记入 `version_source`。**显式补充（文档化 supplement，与 §2 顺序兼容）**：能力承载为独立版本化发布包（npm/crates.io，如 `@42ch/nexus-contracts`）时，以该包自身 manifest 版本为准并直接记入 `version_source`，不套用 workspace 优先级。
- 多 manifest 仓库（Spoke 仓库有 Cargo.toml + Package.swift + go.mod）：每行选一个 manifest 记录；无版本号字段的 manifest（`Package.swift`、`go.mod`）不参与优先级，相关能力回退 workspace manifest。
- Nexus 仓库根部无 `CHANGELOG.md` 属预期（回退 manifest/docs），不是错误。
- 基线文件头部记录：产品名、调研日期、surveyed HEAD SHA（full + short）、版本优先级说明。

## 刷新指引

> **前置条件**：刷新流程需要能访问 Nexus/Spoke 仓库检出。第三方开发者无仓库访问权时**不需要**刷新基线——按 `SKILL.md` 工作流把漂移记入检查清单头部即是对应的处理。

**何时重新调研（触发条件，任一命中即刷新）**：
1. 基线头部记录的 HEAD SHA 与 Nexus/Spoke 仓库检出内 `git rev-parse --short HEAD` 的结果不一致（机械漂移检测）。
2. `version_source` 指向的 manifest 版本号变化（workspace 版本 / npm 包版本 / CHANGELOG 新条目）。
3. 被调研仓库新增/删除公开能力面（新 CLI 子命令、新 HTTP 路由族、新发布包、新协议信封）。
4. 本目录 README 的 `<area>` 词表需要新增 area（先评审后加）。

**刷新步骤**：
1. 只读重跑本 README §调研方法 的来源清单（仍是文档/manifest/公开面，不读 crate 实现）。
2. 更新受影响行：`version`、`version_source`、`source_path`、`survey_date`（= 新调研日）与新 HEAD SHA；未变行保留原 `survey_date` 与旧 SHA 记录即可（头部记录"最近一次全量调研"）。
3. 语义变化的行遵循 id 不可变性：内容重定义 → 新 id，旧 id 在本 README 标 deprecated + 指向后继。
4. 重跑自检（见下），并在变更提交说明中列出漂移比对（旧 SHA/版本 → 新 SHA/版本）。

**机械漂移检测命令**（供刷新或定期核查；在对应仓库检出内执行）：

```bash
# Nexus 仓库检出内执行；与 nexus-capabilities.md 头部比对
git rev-parse --short HEAD
# Spoke 仓库检出内执行；与 spoke-capabilities.md 头部比对
git rev-parse --short HEAD
```

**自检（每次落盘必跑，grep 可复核）**：
1. id 全匹配 regex：提取两文件 id 列后逐条比对 `^(nexus|spoke)\.[a-z][a-z0-9-]*\.[a-z][a-z0-9-]*$`。
2. 跨文件唯一：全部 id `sort | uniq -d` 为空。
3. 每行含 `version` + `version_source` + `survey_date`（规范列）；`research_date`（deprecated 别名）如仍存在须与 `survey_date` 同值。
4. 行首 `product` 与 id 前缀一致；无 `partial` 状态词；无越调研边界内容。

## Deprecated ids

（当前无。规则：`旧 id → 后继新 id`，供 nexus-feedback 逐字复用。）

## Open questions（有界调研边界内未确认项）

- `nexus.sdk.compute-module-abi` 与 `nexus.config.strategy-bundle` 的规范版本号不在任何 manifest 中（ABI 规范/`preset.version` 为独立体系），基线以 workspace `0.1.0` 为 fallback；若未来需要独立版本跟踪，需在刷新时单独核对仓库内部规范（非公开基线证据，如 `specs/compute-module-abi.md`；**不在有界调研范围内，不作基线证据来源**）与 `docs/strategy-authoring.md` 的修订历史。
- Nexus 仓库的 `schemas/daemon-api/` 目录持续新增路由族（survey 时点见 `schemas/README.md` 计数）；HTTP 面的行级拆分粒度（一行 vs 按路由族多行）由后续 checklist 需求决定，当前保持一行 + description 枚举。
- Spoke 原生绑定发布通道（NuGet/Maven/SPM/Go/PyPI）的版本号只能从 spoke-connect workspace 版本回推（`Package.swift`/`go.mod` 无版本字段）；若绑定通道版本与 lockstep 版本脱钩，需在刷新时注明。
