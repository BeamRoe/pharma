# pharma

制药质量管理（GMP / QC）方向的 AI skill 集合。每个 skill 是一组可复用的任务指令，供 AI 助手在处理具体质量事件时加载并遵循。

## 已收录 skill

### 质量事件处理主链（S0 → S5）

以 `pharma-qe-orchestrator` 为编排入口，按节点顺序串联起一条完整质量事件调查链：

| 节点 | Skill | 职责 |
|---|---|---|
| 编排 | [`pharma-qe-orchestrator`](skills/pharma-qe-orchestrator/SKILL.md) | 主链状态机、节点调用表、人工闸门、双出口裁定 |
| S0 | [`pharma-qe-s0-event-intake`](skills/pharma-qe-s0-event-intake/SKILL.md) | 事件登记：把现象描述整理成六要素事件描述并分流 |
| S1 | [`pharma-qe-s1-initial-triage`](skills/pharma-qe-s1-initial-triage/SKILL.md) | 初步调查：Ⅰa／Ⅰb 逐项排查，判定是否已查明根本原因 |
| S2a | [`pharma-qe-s2-investigation-plan`](skills/pharma-qe-s2-investigation-plan/SKILL.md) | 根本原因调查·调查方案（目的/范围/内容三段式） |
| S2b | [`pharma-qe-s2-investigation-report`](skills/pharma-qe-s2-investigation-report/SKILL.md) | 根本原因调查·调查报告成文 |
| S3 | [`pharma-qe-s3-risk-assessment`](skills/pharma-qe-s3-risk-assessment/SKILL.md) | 风险评估：事件直接影响 + 根因体系扩展影响（Critical/Major/Minor） |
| S4 | [`pharma-qe-s4-capa`](skills/pharma-qe-s4-capa/SKILL.md) | CAPA：纠正针对事件本身、预防针对根本原因；五维评估 |
| S5 | [`pharma-qe-s5-selfcheck-gate`](skills/pharma-qe-s5-selfcheck-gate/SKILL.md) | 自检与交付闸门：C 式审核自检 + 三层分离 |

配套 skill：

- [`oos-investigation-review`](skills/oos-investigation-review/SKILL.md) — 审核端：C 式审核报告质量、逻辑合规性、8 维 D0–D7
- [`pharma-report-narrative`](skills/pharma-report-narrative/SKILL.md) — 报告正文叙述体写作（R0–R6 泛化、表格准入、脱表阅读闸门）
- [`pharma-defect-response`](skills/pharma-defect-response/SKILL.md) — 缺陷回复线：自检/审计缺陷的调查与 8 段回复报告
- [`pharma-gmp-gap-scan`](skills/pharma-gmp-gap-scan/SKILL.md) — GMP 体系差距扫描：输出调查方向而非结论
- [`pharma-data-integrity-assessment`](skills/pharma-data-integrity-assessment/SKILL.md) — 历史数据可靠性定量评估（系统稳健性模型）
- [`qc-data-report-kanban-pipeline`](skills/qc-data-report-kanban-pipeline/SKILL.md) — QC 分析报告经审核定稿的流水线

### `qc-defect-analysis` — QC 自检缺陷分析

质量事件趋势与自检缺陷分析（含周期 / 年度回顾）：从台账 Excel 原始数据出发，按责任班组 / 月份 / 缺陷类型多维度分类、根因分层、CAPA 措施与有效性评估、跨期同比与周期回顾、高风险低频事件识别。

- **覆盖数据源**：QC 自检缺陷台账、OOX（OOS / OOT / OOE）、偏差与验证偏差、无效测试台账。
- **默认聚焦**：原辅料 + 包材及水分析组，可按班组筛选。
- **结构**：
  - [`skills/qc-defect-analysis/SKILL.md`](skills/qc-defect-analysis/SKILL.md) — skill 正文（阶段 0–5 全流程、十问分析框架、自检表 D1–D8 / Q1–Q6、输出格式）
  - [`skills/qc-defect-analysis/USAGE.html`](skills/qc-defect-analysis/USAGE.html) — 面向使用者的可视化说明
  - [`skills/qc-defect-analysis/references/`](skills/qc-defect-analysis/references/) — 方法论参考文件（数据模型、解析陷阱、CAPA 有效性、HTML 输出模式、跨公司迁移判据等）
- **触发词示例**：QC自检、缺陷分析、偏差分析、质量趋势、年度回顾、CAPA有效性回顾、self-inspection、defect analysis。

## 使用说明

1. AI 助手命中触发词后，第一步 `skill_view` 加载 `skills/qc-defect-analysis/SKILL.md` 正文，不要凭记忆直接开跑。
2. 按正文「阶段 0–5」执行：文件可访问性检查 → 列结构探测与口径界定 → 数据解析与清洗 → 多维分析 → 报告生成（含自检表）。
3. 报告必须自带 D1–D8（数据正确性）+ Q1–Q6（分析质量）自检表，缺此节视为未完成。

## 迁移到其他公司 / 环境

`references/portability-and-migration.md` 给出 references 三类可移植性判定（通用方法 / 举例带公司数据 / 绑死本公司）与一次执行适配新环境的流程。

## 说明

- 本仓库为公开仓库。上传前已清理 skill 中硬编码的本机文件系统路径与本机项目引用（如 `~/文档/…`、`/home/openclaw/…`、内部审核意见文件名），方法论与举例保留。
- skill 正文在 `SKILL.md`「已知陷阱」「版本管理」等节中部分举例仍含年份专属经验（哪年哪个 sheet 第几列），换数据源后需按 `portability-and-migration.md` 的判据下沉为可迁移方法。

### 质量事件主链各 skill 的发布策略

主链 S0–S5 及配套 skill 本次**仅发布 `SKILL.md` 正文**：

- `references/` 下的方法论参考文件、判据库、案例库（case study）与脚本 **未包含在本仓库**，因其中含公司内部台账结构、真实审核记录与物料信息。
- 正文因此存在指向未发布文件的引用（如 `references/gate_checklist.md`）。这些引用在正文里以「单一存放处，引用不复制」的跨 skill 形式出现，迁移时需按各 skill 的正文指引自行补齐对应 references。
- 正文中真实产品/物料名、批号与事件编号已做通用化处理（如「胶塞水分」→「包材水分」、事件编号→`123456`），仅保留方法论示例结构。
- 落盘路径统一改为占位符 `<质量事件根目录>/`，不绑定具体环境。

若需在其他环境完整运行主链，请从公司内部渠道获取对应的 `references/` 文件，并按正文中的 references 引用关系归位。
