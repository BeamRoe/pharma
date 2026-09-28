# pharma

制药质量管理（GMP / QC）方向的 AI skill 集合。每个 skill 是一组可复用的任务指令，供 AI 助手在处理具体质量事件时加载并遵循。

## 已收录 skill

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
