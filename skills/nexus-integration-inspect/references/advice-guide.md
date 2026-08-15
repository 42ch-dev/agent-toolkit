# 建议编写规则 — 集成建议与自研建议

本文件规定 `nexus-integration-checklist.md` 中两类建议的编写规则：

- **集成建议（integration advice）**：让开发者用**现有** Nexus/Spoke 能力补上未集成项。
- **自研建议（self-dev advice）**：给**基线未覆盖**、被检项目特有的需求指方向（独立章节，不是行级建议）。

## 集成建议（not-integrated 行必写）

**适用**：`not-integrated` 且能力适用的行（`advice_type: integration`）。

内容必须基于能力基线的该行，至少包括：

1. **接入面（integration surface）**：按基线行描述（CLI / HTTP / SDK / agent 接口 / 协议信封 / 配置格式）。
2. **版本与来源**：基线行的 `version` + `version_source`（如 `@42ch/spoke-schemas` npm 包 `0.10.0`、`spoke-connect` crate `=0.10.0`）。
3. **示例 / 参考位置**：把基线行 `source_path`（调研检出相对路径）转换为**可移植引用**——仓库限定相对路径 + 基线头部的调研上下文（如「Nexus 仓库 `docs/nexus-runtime.md`（surveyed HEAD `7b04deaf`）」、「Spoke 仓库 `packages/spoke-connect-ts/`（npm 包 `@42ch/spoke-connect` 0.10.0）」），或可公开访问的 URL（npm / crates.io / GitHub 路径）。**不要**照抄 `source_path` 原值——调研检出相对路径以 `..` 开头、只对调研检出有效，对第三方被检项目（输出清单写入其工作区根）通常解析不到。给出一条可执行的接入路径——具体到命令 / 函数 / 请求面，而不是泛泛而谈。

**禁止**：

- 编造基线之外的 API、字段、端点；建议只能引用基线行 description / `source_path` 对应的公开面。
- 用「请联系维护者」之类的空话代替可执行步骤。
- 替被检项目改代码（本技能只给建议，不修改被检项目）。

## 自研建议（unlisted，独立章节）

**适用判据（两者都满足才进自研建议区）**：

1. **基线未覆盖**：需求在能力基线（`references/capabilities/nexus-capabilities.md` + `references/capabilities/spoke-capabilities.md` 的全部行）中无匹配 capability id——含跨产品检索后仍无匹配。
2. **被检项目特有**：是该项目的真实需求；若疑似基线调研遗漏（而非项目特有），提示用户走反馈流程登记，而不是现场发明 id。

**写法**：

- 每条 = 需求描述（一句）+ 建议的自研方向（数据模型 / 接口 / 接入点层面的方向性建议）。
- **不得**为 unlisted 需求虚构 capability id；自研建议区不产生行级 `advice_type`。
- 自研建议**不是** `not-integrated` 行集成建议的替代品：需求已有能力可满足 → 属于集成建议；两者可并存（部分用现有能力 + 部分自研），但**章节分开**。
- **不得静默丢弃**：发现 unlisted 需求必须写入该区。

## 语气规则

- 中性、可执行、具体；证据导向——建议与基线行 `version` / `source_path` 挂钩。
- 不评判被检项目的选择；不承诺集成后一定可用（建议指向公开面，验证由开发者做）。
- 不夸大基线覆盖范围：基线未覆盖的明确说「基线未覆盖」。

## not-applicable 一行理由示例

- 「项目为纯 TS 前端集成，无 wasm32 计算模块（nexus.sdk.compute-module-abi）。」
- 「项目不通过外部策略包驱动调度，策略内置于自家服务（nexus.config.strategy-bundle）。」
- 「项目仅消费 Spoke 数据协议，无需 C#/Kotlin/Swift/Go/Python 原生绑定（spoke.sdk.native-bindings）。」

## 代表性示例（partial worked example）

三种状态的示例行 + 一条集成建议 + 一条自研建议的代表性示例，见 `references/example-checklist.md`（假想项目「my-agent-demo」的代表性输出示例）；`references/checklist-template.md` §示例行为精简版行示例。**运行时输出必须包含全部基线行（当前 21 行）**；示例文件仅演示状态与建议的写法，不代表输出行数。
