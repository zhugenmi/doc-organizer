## Description: <br>
面向项目的文档组织与内容规范。覆盖：按项目类型（全栈/CLI/库/数据管线/ML 服务）裁剪目录结构、日期优先命名（`YYYY-MM-DD-事宜.md`）、边界声明（每个目录一条互斥规则）、重复文档合并、README 导航路由、跨引用修复，以及四类文档的内容模板（架构/修复/ADR/runbook）。架构模板覆盖背景目标、模块划分、运行机制（生命周期/数据流/状态机/并发/失败处理）、接口契约、技术栈表（链接 ADR）、已知限制；修复模板覆盖问题背景、症状、根因分析（排查路径 + 直接原因 vs 根本原因）、解决方案、验证、复盘与教训。拓宽后的 `fix/` 边界涵盖代码 bug + 性能 + 集成 + 部署 + 数据问题，与 `eval/bad-cases/`（模型输出质量问题）按"修复在哪一层"区分。技术栈总览（arch，现状描述）与选型决策（adr，论证）分离避免副本漂移。 <br>

This skill is ready for commercial/non-commercial use. <br>

## Publisher: <br>
[zhugenmi](https://github.com/zhugenmi) <br>

### License/Terms of Use: <br>
MIT-0 <br>

## Use Case: <br>
开发者用于：新项目搭建 docs/ 骨架；现有散乱 docs/（10+ flat 文件）整理重排；phase 文件重命名为日期-主题格式；按项目类型裁剪目录子集；写架构/修复/ADR/runbook 文档时有内容模板基线；问题分类判断（代码 bug vs 模型 bad case vs 设计决策 vs 监控/运维）。适用于"文档散落找不到"、"想规范文档目录"、"新项目想搭好文档骨架"、"phase 文件太多想按主题分类"等场景。不适用于 <5 文档的小项目（flat 即可）或单一根 README 项目。 <br>

### Deployment Geography for Use: <br>
Global（中英文均可触发，模板与示例中英混排） <br>

## Known Risks and Mitigations: <br>
Risk: `fix/` 边界拓宽后可能误将模型输出质量问题归入 `fix/`。 <br>
Mitigation: 边界声明明确"修复在哪一层"——代码/配置/流程 -> `fix/`，prompt/数据/模型 -> `eval/bad-cases/`；README 边界表与 `references/boundary-declarations.md` 均强调此区分，并给出具体消歧测试。 <br>
Risk: 预创建空目录（`adr/`、`runbooks/`、`eval/bad-cases/`）导致目录腐烂。 <br>
Mitigation: skill 明确"don't pre-create empty dirs"原则，区分"几周内会填"（创建 + `.gitkeep`）和"可能永不填"（README 声明意图，首次使用时创建）；以"是否确信数周内有文档落地"为测试标准。 <br>
Risk: 架构文档可能写成纯图表缺乏运行机制说明，新工程师无法据此排查问题。 <br>
Mitigation: `assets/arch.template.md` 强制要求运行机制章节（生命周期/数据流/状态机/并发/失败处理 5 子项）；SKILL.md 的"Content guidance by doc type"章节明确这是新工程师理解系统的关键，跳过则文档"只是张漂亮的图"。 <br>
Risk: 修复文档可能写成变更记录而非知识沉淀，半年后无人受益。 <br>
Mitigation: `assets/fix.template.md` 强制要求排查路径、直接原因 vs 根本原因、可执行验证步骤、复盘与教训（预防措施 + 可复用经验）；SKILL.md 明确"无复盘的 fix 文档是 changelog 条目，不是修复记录"。 <br>
Risk: 技术栈信息在 `arch/` 和 `adr/` 重复，副本漂移。 <br>
Mitigation: 边界声明区分"现状描述（arch 技术栈表）"和"决策论证（adr）"，arch 技术栈表链接到 ADR 而非复制理由；SKILL.md 的"Common pitfalls"明确禁止重复。 <br>
Risk: 文档重命名后跨引用断裂，比原状态更糟。 <br>
Mitigation: 工作流第 8 步强制 grep 旧路径并修复，修复后重跑 grep 验证零残留；SKILL.md 将"忘记修复引用"列为常见陷阱。 <br>
Risk: 日期取自文件 mtime 而非文档内容，导致时间线失真。 <br>
Mitigation: 工作流第 1 步明确"先从文档正文提取日期（`日期：`、`> Date:`、frontmatter），mtime 仅作兜底"；SKILL.md 原则"Dates first, slugs second"强调此点。 <br>

## Skill Output: <br>
**Output Type(s):** [Text, Markdown, Guidance] <br>
**Output Format:** [Markdown 文档结构方案 + README.md 导航路由 + 各类文档内容模板（arch/fix/adr/runbook）+ 重命名/合并方案 + 问题分类判断 + 边界声明表] <br>
**Output Parameters:** [1D] <br>
**Other Properties Related to Output:** [以 Markdown 为工作格式；输出含目录树、边界声明表、按场景路由区；模板覆盖架构设计/模块划分/运行机制/技术栈/bug 修复/问题解决；支持中英文混排；模板含占位符注释引导填写] <br>

## Skill Version(s): <br>
1.0.0 <br>

## Ethical Considerations: <br>
用户应评估本 skill 是否适合自身项目规模（<5 文档的项目 flat 即可，无需本 skill）；整理文档时注意保留原始内容，合并时用 `---` 分隔保留全部正文并加 provenance 行（`> 合并自 X + Y`），不删除内容只重构容器；日期应取自文档内容而非 mtime，避免时间线失真；跨文档引用修复后需 grep 验证无残留旧路径；不要为了"减少 clutter"合并不相关文档，合并应基于相关性而非数量；预创建目录前用"数周内是否确信有文档落地"自测，低置信度时仅在 README 声明意图。 <br>

## Test Results: <br>

**测试配置**：3 个测试用例 × 2 配置（with_skill / without_skill）= 6 runs，每配置 1 run。 <br>

**总体结果**： <br>

| 指标 | with_skill | without_skill | Delta |
|------|-----------|---------------|-------|
| 通过率 | 100.0% ± 0.0% | 31.8% ± 14.4% | **+68.2pp** |
| 耗时（秒） | 69.4 ± 11.5 | 44.7 ± 5.6 | +24.7s |

**分测试用例**： <br>

| 测试用例 | with_skill | without_skill | Delta |
|---------|-----------|---------------|-------|
| eval-4 架构文档内容指引 | 7/7（100%） | 2/7（29%） | +71pp |
| eval-5 修复文档内容指引 | 6/6（100%） | 1/6（17%） | +83pp |
| eval-6 问题分类判断 | 8/8（100%） | 4/8（50%） | +50pp |

**关键发现**： <br>
1. **内容深度是核心价值**（eval-5 delta 最大，+83pp）：skill 强制要求排查路径、直接原因 vs 根本原因、复盘与教训；无 skill 只写"根因 + 修复"，本质是变更记录而非知识沉淀。 <br>
2. **边界精度决定分类正确性**（eval-6）：无 skill 将性能问题误归 `monitor/`、部署问题误归 `runbooks/`；skill 用"修复在哪一层"原则正确归入 `fix/`。 <br>
3. **架构文档结构化**（eval-4）：skill 产出模块表 + 运行机制（5 子项）+ 技术栈表（链接 ADR）+ 已知限制；无 skill 缺模块表、技术栈、已知限制。 <br>
4. **时间成本可接受**：with_skill 多花 24.7s 用于读 skill 文件与模板，换来 68pp 质量提升。 <br>

