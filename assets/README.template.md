# {{Project}} 文档总览

> Replace {{Project}} with the project name. Keep the intro to one line - this file is a router, not an introduction.

本目录存放 {{Project}} 项目的技术文档。按类型分目录，文件名 `日期-事宜.md`，便于按时间线与主题检索。

## 📌 各目录边界声明

> 这个表是契约。每个目录有且只有一条规则，规则之间互斥。新增文档先查表定归属。

| 内容 | 归属目录 |
|------|----------|
| 技术设计 / 模块划分 / 运行机制 / 技术栈总览 | `arch/` |
| 架构决策记录（为什么选 X 不选 Y） | `adr/` |
| 代码 Bug / 性能 / 集成 / 部署 / 数据问题修复 | `fix/` |
| 模型 Bad Case 分析（修复在 prompt/数据/模型） | `eval/bad-cases/`（按需建立） |
| 评测方案 / 测试报告 | `eval/` |
| 成本 / 日志 / 监控指标定义 | `monitor/` |
| 故障应急操作手册（指标异常时怎么办） | `runbooks/` |
| 人机协作流程 / Issue triage / 领域分工 | `agents/` |

**关键区分**：
- `arch/` 是**技术视角**（系统怎么跑、模块怎么分、运行机制是什么、技术栈是什么）；`agents/` 是**人员视角**（团队怎么协作）。
- `arch/` 的技术栈表只列**当前在用什么**并链接到 ADR；`adr/` 记录**为什么选它**。前者是现状描述，后者是决策论证，不要重复。
- `fix/` 是**工程侧问题**（代码/配置/流程修复）；`eval/bad-cases/` 是**模型输出质量问题**（prompt/数据/模型修复）。区分标准是"修复在哪一层"，不是症状严重程度。
- `monitor/` 告诉你**看什么**指标；`runbooks/` 告诉你指标异常时**怎么处理**。

## 目录结构

```
docs/
├── README.md        # 本文件，文档导航
├── arch/            # 系统设计（技术视角）
│   ├── overview/    # 顶层架构、工作流编排、跨模块系统
│   ├── pipeline/    # 管线各 Stage 设计
│   ├── client/      # 前端与 BFF 集成
│   └── api/         # 对外接口契约
├── adr/             # 架构决策记录（NNNN-事宜.md）
├── monitor/         # 日志格式、监控指标、token 成本追踪
├── eval/            # 评测方案与测试报告
│   └── bad-cases/   # 模型 Bad Case 分析（按需建立）
├── fix/             # 代码/性能/集成/部署/数据问题修复记录
├── runbooks/        # 故障应急操作手册
└── agents/          # 协作规范（Issue triage / 领域文档，人员视角）
```

> 按项目类型裁剪。CLI 工具不需要 `client/`；库不需要 `monitor/`/`runbooks/`。见 `references/project-types.md`（在 skill 内）。

## 按场景找文档

### 我想了解整体架构 / 模块设计 / 运行机制

**顶层设计与编排**（`arch/overview/`）：
- [`overview/YYYY-MM-DD-*.md`](arch/overview/) - 一句话描述。架构文档应包含：背景目标、模块划分、运行机制（生命周期/数据流/状态机/并发/失败处理）、接口契约、技术栈表。

**管线各 Stage**（`arch/pipeline/`）：
- [`pipeline/YYYY-MM-DD-*.md`](arch/pipeline/) - 一句话描述。

**前端与集成**（`arch/client/`）：
- [`client/YYYY-MM-DD-*.md`](arch/client/) - 一句话描述。

**接口契约**（`arch/api/`）：
- [`api/YYYY-MM-DD-*.md`](arch/api/) - 一句话描述。

### 我想看技术选型 / 为什么选 X 不选 Y

**架构决策记录**（`adr/`）：
- [`adr/0001-*.md`](adr/) - 一句话描述决策。每个 ADR 应包含：背景、决策、理由（含备选方案与拒绝原因）、后果。

**当前技术栈总览**：见 `arch/overview/` 下的架构文档中的技术栈表，链接到各 ADR。

### 我想查日志格式 / 监控指标

- [`monitor/YYYY-MM-DD-*.md`](monitor/) - 一句话描述。

### 指标异常时怎么办

- [`runbooks/YYYY-MM-DD-*.md`](runbooks/) - 一句话描述触发场景。每个 runbook 应有症状速查表 + 排查步骤 + 验证 + 升级条件。

### 我想跑评测 / 看成本数据

- [`eval/YYYY-MM-DD-*.md`](eval/) - 一句话描述。

### 我遇到了问题 / 想看历史问题怎么解决的

**修复与问题记录**（`fix/`）：
- [`fix/YYYY-MM-DD-*.md`](fix/) - 一句话描述。修复文档应包含：问题背景、症状、根因分析（含排查路径）、解决方案、验证、复盘与教训。

> `fix/` 不只是代码 bug。性能回退、集成失败、部署中断、数据问题都算--只要修复在代码/配置/流程层，就放这里。模型输出质量问题（修复在 prompt/数据/模型层）放 `eval/bad-cases/`。

### 我要参与工程协作 / 提 issue

- [`agents/*.md`](agents/) - 一句话描述。

## 约定

- **命名**：`YYYY-MM-DD-事宜.md`。日期取自文档内容；无法确定时用最近日期。日期在前让 `ls` 输出即时间线。
- **ADR 编号**：`adr/NNNN-事宜.md`，四位序号，递增不复用。
- **内容基线**：每种文档类型在 skill 的 `assets/` 下有模板（arch / fix / adr / runbook）。新文档以模板为基线，不是可选。文档的价值在内容深度，不在数量。
- **合并**：内容高度重叠或前后承接的文档已合并（标题注明「合并自 X + Y」），正文以 `---` 分隔保留全部内容。
- **跨文档引用**：使用 `docs/<dir>/<file>.md` 相对路径，便于 grep 与跳转。
- **新增文档归属**：先查边界声明表，再决定目录。不确定时优先 `arch/`，明确是「为什么选 X」入 `adr/`，明确是「修了问题」入 `fix/`。
