---
name: oos-investigation-review
description: 审核端（非事件入口）：C 式审核已起草的 QC 报告与调查报告——报告质量、逻辑合规性、8 维 D0–D7。事件调查入口见 pharma-qe-orchestrator。
version: 0.7.0
author: Hermes
trigger_keywords:
  - "审核报告"
  - "报告审核"
  - "审核调查报告"
  - "审核这份报告"
  - "C式审核"
  - "报告质量审核"
  - "终审"
metadata.hermes.tags: OOS, Investigation, Quality Event, Root Cause, CAPA, Narrative Logic, Multi-Type
---

# OOS Investigation Report Review — C-Style Audit Framework

## Purpose

When an investigation report or technical document has been drafted (by B-level personnel), apply a structured review checklist modeled on department head C's actual review approach. This skill supports **multiple document types** with type-specific dimension weights and check criteria.

## Core Principle: Procedure Precedence（程序优先原则）

在GMP框架下，检验结果的可靠性建立在检验程序受控的基础上。调查的逻辑顺序必须是：

1. **确认程序合规**：试剂状态、仪器校准、方法执行、记录完整性
2. **以程序异常为调查突破口**：若发现程序偏离，追溯根本原因
3. **评估结果有效性**：仅在程序受控的前提下，评价结果可靠性

**禁止逻辑倒置**：
- ❌ 用"结果看起来合理"证明"程序没问题"（举证责任倒置）
- ❌ 用"即使程序有问题，结果也可靠"替代程序合规性调查
- ❌ 用技术计算覆盖管理调查义务

## 审核边界声明

本skill审核的是调查报告的**叙事质量**和**逻辑合规性**，而非调查本身的充分性。

| 审核范围 | 不审核范围 |
|---------|-----------|
| 报告中是否合理呈现调查结果 | 调查表单是否完整填写 |
| 报告逻辑是否符合GMP原则 | 调查过程是否逐条确认 |
| 根本原因是否追溯到管理机制 | 证据附件是否齐全 |

审核者不应追问"调查表在哪里"，而应关注"报告中是否恰当浓缩了调查结论"。前者是流程问题，后者是叙事问题。

## Triggers

- User provides a report (.docx, .md, or pasted text) and asks for review/audit
- User says "审核", "review", "检查报告质量", "看看这份报告有什么问题"
- User wants to evaluate an investigation before submitting to management

## Supported Document Types

| Type | Description | Weight Vector |
|------|-------------|---------------|
| `oos` | OOS/OOT investigation report | [1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0] |
| `deviation` | General deviation / validation deviation | [1.2, 1.0, 0.8, 1.0, 1.0, 1.0, 1.2, 1.0] |
| `pharmacopoeia-comparison` | ChP/USP/EP pharmacopoeia comparison | [0.6, 0.0, 1.0, 1.2, 1.0, 0.0, 0.6, 0.0] |
| `sop-revision` | SOP/procedure revision assessment | [0.8, 0.5, 0.4, 0.8, 1.2, 0.8, 0.8, 0.5] |
| `work-summary` | Team work summary / annual report | [0.6, 0.8, 0.8, 0.4, **1.0**, 1.0, 0.6, 0.4] |
| `change-control` | Change control impact assessment | [1.0, 1.0, 1.0, 1.2, 1.0, 1.0, 1.0, 1.0] |
| `quality-event` | Quality event / deficiency report | [1.2, 1.2, 0.8, 1.0, 1.0, 1.0, 1.2, 1.2] |

审核时参考 `references/pharmacopoeia-comparison-checklist.md` 获取完整checklist。

If user doesn't specify type, default to `oos`.

## What This Skill Does

Applies an 8-dimension audit to any investigation or technical report (6 original + 1 from 2026-08-28 case study + 1 from 2026-08-29 落点分析):

1. **叙事顺序** (D0) — Is the narrative in objective chronological order (time-linear: preparation→use→packaging→storage→observation) rather than defensive "I discovered" style? Must include complete event portrait.
2. **因果链完整性** (D1) — Does the report connect evidence → inference → conclusion with explicit logical links? Must split direct cause and root cause into separate layers.
3. **根因深度** (D2) — Does the root cause go beyond "phenomenon stacking" to systemic management failure? Must追问评估/考察机制为什么没有覆盖到这个风险.
4. **数据桥接** (D3) — Are disparate data points connected into a single evidence chain with quantitative bridges?
5. **风险表述严谨性** (D4) — Does the report distinguish "proven fact" from "reasonable inference"? Must avoid defensive self-proving narratives.
6. **GMP框架嵌入** (D5) — Does the report reference relevant SOPs, CAPAs, pharmacopoeia clauses, and quality system principles?
7. **行动项分层** (D6) — Are corrective/preventive actions layered (short/mid/long term) rather than a single item? Actions must be verifiable with clear deliverables.
8. **风险评估落点** (D7) — Does the risk assessment's conclusion land on "结果的可靠性" (评估：已用批次结果可靠、无需复验) rather than "缺陷的影响" (减责：影响可忽略、风险较低)? Must cover the progressive-change window period (渐变窗口期).

## Dimension Definitions & Type-Specific Criteria

### D0: 叙事顺序 (Narrative Order)

| Score | Criteria |
|-------|----------|
| 0 | Narrative is defensive ("I discovered..."), lacks event portrait, jumps to findings without background |
| 1 | Has some background but narrative order is non-linear or incomplete |
| 2 | Time-linear narrative complete: preparation→use→packaging→storage→observation. No defensive self-proving language. |

**quality-event 专属操作法 — 检查缺陷/缺陷描述二分**（源：异常峰缺陷 V3.0 案例）：

外部审计/缺陷回复类文档，结构上须把两件事拆开，不得混写：
- **"检查缺陷"（客观现场发现）**：只陈述检查当日调取了哪些记录/图谱/物料、发现了什么客观异常，不含任何归因、不含缺陷定性。
- **"缺陷描述"（完整事件画像）**：按时间线性展开事件全貌（检测活动→异常出现→判定过程→未发起事件），并在画像中显式写出"检验员及审核人员未识别、未发起OOE"这类程序性事实——这是缺陷的定性核心。

审核时若发现"检查缺陷"段已夹带归因（如"因SOP未收载图谱"）或"缺陷描述"仍停留在"现场调取发现"的防御性开场，按 D0=1 处理。

### D1: 因果链完整性 (Causal Chain Completeness)

| Score | Criteria |
|-------|----------|
| 0 | No logical chain. Events listed chronologically without connections. Reader must infer causality. |
| 1 | Some logical links present but gaps exist. E.g., data presented but no "therefore" statement connecting to next step. |
| 2 | Explicit evidence → inference → conclusion chain. Each section ends with a bridge to the next. 缺陷描述必须按时间线性展开（配制→使用→包装→储存→现象），不得从"现场检查发现"切入。直接原因部分必须从试剂理化性质出发推演作用机理，并包含对比论证（如"为什么A发生但B未发生"）。 |

**Type-specific adjustments:**
- `oos` / `deviation`: Focus on investigation flow (Ia→Ib→root cause→CAPA). Missing link = major gap. **必须拆分为直接原因和根本原因两层，不得合并。**
- `pharmacopoeia-comparison`: Replace with **差异分类完整性** — Are all differences classified correctly (substantive vs textual vs numbering)? Is there a clear structure for presenting differences?
- `sop-revision`: Replace with **修订逻辑清晰度** — Is the reason for change clear? Are before/after states documented?
- `work-summary`: Replace with **趋势判断深度** — Are numbers just listed, or is there structural insight about what changed and why?
- `change-control`: Focus on change rationale → impact analysis → risk mitigation chain.

### D2: 根因深度 (Root Cause Depth)

| Score | Criteria |
|-------|----------|
| 0 | Root cause = phenomenon description ("X happened because Y"). No system-level insight. |
| 1 | Root cause mentions process/system factor but doesn't trace back to management gap. |
| 2 | Root cause identifies systemic management failure (e.g., "method change without standard applicability assessment," "no impact evaluation after procedure update"). **必须追问：评估/考察机制为什么没有覆盖到这个风险？** 不能停在"缺规定"层面。 |

**Type-specific adjustments:**
- `oos` / `deviation`: Full weight. Must reach systemic level.
- `pharmacopoeia-comparison`: **Not applicable (score=0 by default)**. Pharmacopoeia comparisons are descriptive, not investigative.
- `sop-revision`: Partial weight. Root cause matters less than whether the revision addresses the identified gap.
- `work-summary`: Partial weight. Look for whether trends are attributed to root causes or just described.
- `change-control`: Full weight. Must assess if change rationale addresses underlying need.

**根因上溯操作法（五级归因链 + 三分排除法）：**

判断根因是否停在 L3「规定内容」而非上溯到 L4/L5「规定来源/管理环节」，用五级链定位：

L0 现象 → L1 机理 → L2 直接原因 → L3 规定内容 → L4 规定来源 → L5 管理根因

「缺规定」= L3 症状，不是根因。上溯 L4/L5 用三分排除法：
1. 先核实：这条规定到底怎么来的（考察 / 评估 / 未评估）？
2. 三分排除：完成了考察 → 考察方法为何没识别 → 考察缺陷；实为评估 → 评估为何不合理 → 评估缺陷；未评估 → 新试剂引入评估环节缺失。
3. 收敛：无论哪个分支，根因都是「前置评估/考察环节缺失或失效」，而非「规定内容缺某条」。

### D3: 数据桥接 (Data Bridging)

| Score | Criteria |
|-------|----------|
| 0 | Multiple data sets presented independently. No connection between them. |
| 1 | Some comparison between data sets but no quantitative bridging. |
| 2 | Disparate data connected via quantitative relationships (e.g., "method A result X ≈ method B result Y," "equivalent to historical batch Z"). |

**Type-specific adjustments:**
- `oos` / `deviation`: Critical. Different testing methods, batches, time periods must be linked. **证据类型须覆盖"历史同类检测回顾 + 涉事批次复测排除"**：把"历史同类记录中未再出现同类异常"与"该批物料再次检测未复现"作为独立排除项写入因果链，用于排除物料/对照品本身质量变化（源：异常峰缺陷 V3.0 案例）。
- `pharmacopoeia-comparison`: **Critical**. ChP/USP/EP values, methods, limits must be cross-referenced and convertible.
- `sop-revision`: Lower priority. Focus on whether revised content is internally consistent.
- `work-summary`: Moderate. Year-over-year and month-over-month data should connect logically.
- `change-control`: Important. Before/after states must be quantified where possible.

### D4: 风险表述严谨性 (Risk Expression Rigor)

| Score | Criteria |
|-------|----------|
| 0 | Inferences stated as facts ("可以确定", "证明"). No boundary marking. |
| 1 | Some use of hedging language but inconsistent. |
| 2 | Clear distinction between proven facts ("数据显示") and reasonable inferences ("推测", "可能"). Conclusion includes uncertainty acknowledgment. |

**Type-specific adjustments:**
- `oos` / `deviation` / `change-control`: **Critical**. Overstating certainty in investigations is a compliance risk.
- `pharmacopoeia-comparison`: **Highest weight**. Most common error: over-interpreting textual differences as substantive changes.
- `sop-revision`: Moderate. Focus on whether revision scope is clearly bounded.
- `work-summary`: Low. Management summaries allow more interpretive language.

### D5: GMP框架嵌入 (GMP Framework Integration)

| Score | Criteria |
|-------|----------|
| 0 | No regulatory references. Investigation exists in a vacuum. |
| 1 | Some references (SOP numbers, pharmacopoeia chapters) but not integrated into reasoning. |
| 2 | Relevant GMP principles explicitly cited and used to frame the investigation (e.g., "method change requires standard applicability assessment per GMP principle," "reference to CAPA-XXX shows prior recognition of this issue"). **当判定某标准/方法不适用时，须检查是否需同步修订关联的内控标准、厂家标准或上下游文件（联动修订）。** |

┌─ 子项 D5.1：方法来源区分（条件触发） ─┐
│ 触发条件：报告引用方法学论证（稳健性/过量设计）时                    │
│                                                                       │
│ 药典方法 → 引用既有稳健性，不得声称"设计"                            │
│ 自建方法 → 引用验证报告章节                                          │
│ 内控方法 → 确认建立依据                                              │
│                                                                       │
└─────────────────────────────────────────────────────────────────────┘

**Type-specific adjustments:**
- `oos` / `deviation` / `sop-revision` / `change-control`: **Critical**. Must cite relevant SOPs, CAPAs, pharmacopoeia, GMP principles.
- `pharmacopoeia-comparison`: **Critical**. Must reference specific pharmacopoeia chapters, official documents, regulatory guidance.
- `work-summary`: **Critical in pharma QC context**. Even trend reports and data analyses MUST embed GMP framework references — cite relevant SOPs, pharmacopoeia clauses, CAPA cross-references, and ALCOA++ compliance. A work summary without GMP framework is non-compliant regardless of document type.

### D6: 行动项分层 (Action Item Layering)

| Score | Criteria |
|-------|----------|
| 0 | Single action item or all items are training-only. |
| 1 | Multiple action items but no time-layering (short/mid/long term). |
| 2 | Actions layered across timeframes AND control levels (training → SOP → engineering → system). **措施动词必须具体（修订/更换/增加/执行），禁止使用"梳理/调查/了解/加强/强化/巩固/完善"等模糊动词。每条措施必须有明确的交付物和判断依据。每条措施的责任人/部门必须具体到人（不得留空或仅写"QC/"），且标注计划完成时限。** |

**Type-specific adjustments:**
- `oos` / `deviation` / `change-control`: **Critical**. CAPA must span multiple control levels.
- `quality-event`: **Critical**. 缺陷回复的整改常只有"修订SOP + 培训"两条，易停留单一控制层级；须检查是否覆盖文件/人/系统三层。措施动词须具体，且每条例外都要给出交付物+责任人+完成时限（模糊动词清单见 Pitfall 6）。
- `sop-revision`: Moderate. Implementation plan should include training, transition period, old document recall.
- `work-summary`: Moderate. Improvement suggestions should be prioritized and time-phased.
- `pharmacopoeia-comparison`: **Not applicable (score=0 by default)**. Comparison reports don't generate action items — they inform decisions.

### D7: 风险评估落点 (Risk Assessment Framing)

| Score | Criteria |
|-------|----------|
| 0 | 风险评估的结论落点主语是"缺陷的影响"（典型句式："不会产生实质影响""影响较小""风险较低""不受影响"）；或仅凭"使用前检查无异常"就得出"历史结果可靠"，未覆盖渐变窗口期（已发生但肉眼不可见的早期变化）。 |
| 1 | 技术分析覆盖了历史批次评估，但落点表述不清晰（"影响可控"等模糊表述）；或虽提及渐变窗口期，但无定量/机理兜底。 |
| 2 | 技术分析以条件假设法明确落点于"结果的可靠性"（典型句式："即使氧化至检查不可见程度，显色反应完全性不受影响，已使用批次结果可靠，无需复验"）；且历史批次评估至少包含一条技术兜底（定量计算或机理分析）或明确说明调查检验路径。 |

**检查要点：**

1. **落点主语检查**：风险评估结论句的主语是"缺陷"还是"结果"？前者应改写。同一个化学计算/机理分析，落点主语不同——落点是"缺陷的影响"= 减责，落点是"结果的可靠性"= 评估。
2. **渐变窗口期检查**：缺陷是否渐进性（氧化/降解/吸附/水分迁移）？若是，"使用前检查无异常"只能排除可见异常，必须评估已发生但不可见早期变化的残余风险。
3. **双路径完整性**：若审核人要求"重新配制/调查检验"，终稿必须体现执行或说明不执行的理由；不能只留"检查无异常"一条弱路径。
4. **条件假设法识别**："即使……也……"句式若用于划定边界（结果仍可靠）→ 保留；若用于压缩严重度（缺陷不严重）→ 删除。关键不是工具本身，而是落点。
5. **落点与整改力度自洽**：影响评估结论与整改力度必须成比例（详见 Pitfall 13）。
6. **视角切换时事实锚点须保留（quality-event 专属，源：异常峰缺陷三版案例）**：版本迭代中"切入角度/审视框架"会切换（如 V1 从"图谱异常+人为漏判"切入、V2 从"SOP是否收载图谱"切入、V3 收拢回图谱主框架并吸收SOP视角），但**缺陷本身的事实（客观发现）不变**。审核要核对的不是"缺陷定义是否前后一致"，而是**事实锚点有没有被视角切换带丢**：
   - 视角可以换，但"检查缺陷"段的客观发现原文必须逐字保留（STD 图谱 2359/2341cm⁻¹ 未识别异常峰这一事实不得被视角改写覆盖）。
   - 若某版视角切换后，原始客观发现被程序性表述（如"因SOP未收载图谱"）替代或淡化，即为**事实锚点丢失**——审核必须打回，要求把客观发现重新写回"检查缺陷"段。
   - 判据：把本版"检查缺陷"段与初稿逐句对齐，凡初稿有的客观事实（图谱编号、波数、人员判定动作）缺失或被归因句替代的，按 D0/D1 缺陷处理。

**Type-specific adjustments:**
- `oos` / `deviation` / `change-control`: **Critical**。风险评估是这些文档的核心，落点错误会直接歪曲评估结论。
- `quality-event`: **Highest weight**。缺陷回复报告最容易出现"减责式"落点。
- `pharmacopoeia-comparison`: **Not applicable (score=0 by default)**。描述性文档，不产生风险评估。
- `sop-revision`: Moderate。修订评估中的影响陈述需检查落点。
- `work-summary`: Low。总结报告中的风险评估较少，但若出现影响评估仍需检查落点。

## Audit Procedure

### Step 1: Discover File Structure

1. Identify the document type (ask user if unclear, default to `oos`)
2. Extract key sections based on document type:
   - **Investigation types** (`oos`, `deviation`, `change-control`): Ia/Ib summary, method background, evidence, conclusion, impact assessment, CAPA
   - **Comparison type** (`pharmacopoeia-comparison`): Difference list, classification, impact evaluation, source references
   - **Revision type** (`sop-revision`): Current state, proposed changes, rationale, implementation plan
   - **Summary type** (`work-summary`): Data overview, trend analysis, findings, recommendations

### Step 2: Apply 8-Dimension Checklist with Type Weights

For each dimension, score 0-2 using the type-specific criteria above, then multiply by the weight vector.

Weight vectors:
```
oos:                 [1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0]
deviation:           [1.2, 1.0, 0.8, 1.0, 1.0, 1.0, 1.2, 1.0]
pharmacopoeia-comparison: [0.6, 0.0, 1.0, 1.2, 1.0, 0.0, 0.6, 0.0]
sop-revision:        [0.8, 0.5, 0.4, 0.8, 1.2, 0.8, 0.8, 0.5]
work-summary:        [0.6, 0.8, 0.8, 0.4, 1.0, 1.0, 0.6, 0.4]
change-control:      [1.0, 1.0, 1.0, 1.2, 1.0, 1.0, 1.0, 1.0]
quality-event:       [1.2, 1.2, 0.8, 1.0, 1.0, 1.0, 1.2, 1.2]
```

Calculate weighted score:
```
weighted_score = Σ(dim_score × weight) / Σ(weights)
```

### Step 3: Generate Review Output

Produce a structured review report:

```markdown
# [Report Title] — 审核评估报告 V[version]

## 审核信息

- **文档类型**: [oos / deviation / pharmacopoeia-comparison / sop-revision / work-summary / change-control / quality-event]
- **审核维度**: 8维C式审核框架（含D0叙事顺序与D7风险评估落点）
- **加权总分**: X/16 (原始分 Y/16)

## 总体评分

| 维度 | 原始分 | 权重 | 加权分 | 评级 |
|------|--------|------|--------|------|
| D0 叙事顺序 | X/2 | W0 | X×W0/2 | 🔴🟡🟢 |
| D1 因果链完整性 | X/2 | W1 | X×W1/2 | 🔴🟡🟢 |
| D2 根因深度 | X/2 | W2 | X×W2/2 | 🔴🟡🟢 |
| D3 数据桥接 | X/2 | W3 | X×W3/2 | 🔴🟡🟢 |
| D4 风险表述严谨性 | X/2 | W4 | X×W4/2 | 🔴🟡🟢 |
| D5 GMP框架嵌入 | X/2 | W5 | X×W5/2 | 🔴🟡🟢 |
| D6 行动项分层 | X/2 | W6 | X×W6/2 | 🔴🟡🟢 |
| D7 风险评估落点 | X/2 | W7 | X×W7/2 | 🔴🟡🟢 |
| **加权总分** | | **ΣW=** | **X.X/16** | **等级** |

评级标准：加权<0.5=🔴 需重写, 0.5-1.0=🟡 需补充, >1.0=🟢 合格

## 逐项审查

### D1: [维度名称] — X/2 (权重: W)
**发现：** [具体引用报告原文说明问题]
**建议：** [具体修改方向]

...

## 必须修改项（红色）
1. [具体问题] — [为什么必须改]
2. ...

## 建议改进项（黄色）
1. [具体问题] — [为什么建议改]

## 亮点（保留）
1. [报告中做得好的部分]
2. ...

## 修订后复核建议
[如果报告修改后，哪些维度需要重点复核]
```

### Step 4: Provide Specific Revision Guidance

For each dimension scoring below the threshold, provide:
- **Quote the exact passage** from the report that is problematic
- **Explain what's wrong** (using C's actual review language when possible)
- **Provide a concrete rewrite suggestion** — not vague advice, but an actual rewording

## Output Modes

Two operating modes — user chooses which:

### Mode A: Audit Only (Default)
You provide the 8-dimension review report with scoring, red/yellow items, and specific rewrite suggestions. The user or B-level author makes the changes themselves.

**Use when:** Building team capability, audit trail required, user wants to control edits.

### Mode B: Audit + Direct Revision
After generating the audit report, you apply the corrections and produce a final version. Key principles:
1. **Preserve B's technical analysis body** — C-style upgrade ≠ content rewrite. Keep all original data, testing methods, batch numbers, and analytical conclusions intact.
2. **Restructure only the narrative** — Reorganize sections into causal-chain narrative (evidence → inference → conclusion), add transition paragraphs between sections.
3. **Deepen root cause** — Replace phenomenon stacking ("X happened because Y") with systemic attribution ("process gap in X led to Y").
4. **Quantify bridges** — Add explicit quantitative relationships between disparate data sets (e.g., "Method A result X ≈ Method B result Y").
5. **Hedge risk language** — Convert definitive claims ("可以确定", "证明") to inferential ("数据显示", "推测", "可能需进一步确认").
6. **Layer CAPA** — Restructure action items into short/mid/long term + training/SOP/engineering/system levels.
7. **Re-frame risk assessment landing point** — Convert conclusion主语 from "缺陷的影响"（减责）to "结果的可靠性"（评估）。同一技术分析保留，但落点主语改为"结果"（详见 D7）。

**Use when:** Time-critical, user explicitly requests "直接改" or "出终稿".

**Verification before delivery:** After revision, check that (a) all original technical data is preserved, (b) no new factual claims were introduced, (c) the document reads as a single coherent narrative rather than section-by-section corrections.

---

## Example: C's Actual Review Patterns

From the 包材水分OOS case (B初稿 vs C审核稿):

### Pattern 1: 结构重构
- B: 线性排列所有相关数据，读者自行拼凑
- C: 重构为"方法变更历史→差异对比→研究数据→综合推导"，每步为下一步铺路
- 可操作建议: "在对比研究和稳定性数据之间加一段过渡，说明'基于上述对比结果，进一步验证长期趋势'"

### Pattern 2: 根因跃迁
- B: "吸湿+方法差异共同导致"（现象并列）
- C: "方法变更后影响评估不足，标准不适用"（系统归因）
- 可操作建议: "在结论段增加'根本原因不是单次操作失误，而是方法变更流程中缺少标准适用性评估环节'"

### Pattern 3: 数据桥接
- B: 企业方法和药典方法数据各自呈现，数值差异巨大但无换算
- C: "药典方法4.59‰ ≈ 企业方法1‰ → 对应制剂放行0.4%~0.5%"
- 可操作建议: "补充两种方法的等效换算关系，并关联到已放行产品的实际质量数据"

### Pattern 4: 风险表述
- B: "可以确定该批次包材水分结果偏高"
- C: "需要进一步确认导致水分结果偏高的原因"
- 可操作建议: "将确定性表述改为推断性表述，标注'需结合后续Ib调查进一步确认'"

### Pattern 5: 合规框架
- B: 无SOP编号、无CAPA追溯
- C: 引用CAPA-250、M-0059、《中国药典》9623
- 可操作建议: "在方法变更描述处补充CAPA编号和依据的药典章节"

### Pattern 6: 风险评估落点
- B: 6000倍过量计算 → "不会对检测灵敏度产生实质影响"（落点=缺陷的影响，减责）
- C 正确做法（非删除，而是改落点）: 同一计算 → "即使氧化至检查不可见程度，显色反应完全性不受影响，已使用批次结果可靠，无需复验"（落点=结果的可靠性，评估）
- 可操作建议: "保留技术分析，但把结论句的主语从'缺陷'改为'结果'，用条件假设法划边界而非压缩严重度"

### 三种增强模式（选其一）

根据技能本质，选择一种增强方式：

1. **"加一面镜子"**（输出前自检）— 适用于数据分析类技能。让已有的分析结果自己检查自己的逻辑连贯性。例：qc-defect-analysis 在生成报告前执行4项自检。
2. **"加一层过滤网"**（流程扩展）— 适用于文档起草类技能。在结构化输出之上再筛一遍GMP合规性。例：pharma-sop-drafting 新增Step 7合规审查。
3. **"加一张关系图"**（关联分析）— 适用于合规检查类技能。把孤立的检查项串成关联的风险画像。例：pacmp-analysis 新增Step 7综合评估。

## Pitfalls

1. **Don't confuse this with statistical analysis.** This skill reviews the NARRATIVE quality of an investigation report, not the data accuracy. Use `pharma-quality-event-analysis` for spreadsheet-to-statistics work.
2. **Don't rewrite the report for the user.** Provide specific guidance and examples, but let the author make the changes. The goal is skill transfer, not replacement.
3. **Score conservatively.** A report that looks good on surface reading may fail on deeper inspection. When in doubt, score lower and explain why.
4. **Respect domain expertise.** The reviewer may not know the technical details as well as the author. Focus on structure, logic, and compliance — not technical correctness of specific test methods.
5. **Flag missing sections, not just weak ones.** An investigation that omits a required section entirely is more critical than one that handles a section poorly.
6. **Type matters.** Always confirm the document type before applying weights. Using `oos` weights on a `pharmacopoeia-comparison` will produce misleading results.
7. **Zero-weight dimensions are NOT flaws.** When a dimension has weight=0 for a document type (e.g., D2 and D6 and D7 for pharmacopoeia-comparison), it means that dimension is not applicable to this document type — not that the report is deficient.
9. **Batch audit requires explicit dispatch grouping.** When auditing 15+ reports, group by document type and dispatch parallel subagents (one per group). Pharmacopoeia comparisons (10 reports) take ~350s per subagent. Always verify file creation after dispatch — subagents may fail silently or hit token limits. Use consistent date suffix format `_YYYYMMDD`. Original reports must NEVER be overwritten. Audit opinions are always separate files with `_审核意见_` prefix.

10. **False-positive vs false-negative risk in OOS impact assessment — critical pitfall.** When evaluating whether an OOS investigation affects historical batches, you MUST correctly identify the direction of the analytical bias and its impact on misclassification risk:

    - **假阳性（False Positive）**：合格品被判为不合格。偏高效应可使真实含量在99.0%~101.0%之间的批次检测结果>101.0%，触发OOS。后果：检验资源浪费，不影响产品质量。
    - **假阴性（False Negative）**：不合格品被判为合格。偏高效应可使真实含量<99.0%的批次检测结果落入99.0%~101.0%，被判定为合格并放行。**后果：患者安全风险。**
    - **关键逻辑**：偏高效应（检测值 > 真实值）既可能导致假阳性，也可能导致假阴性。不能说"偏高不会掩盖低含量"——恰恰相反，偏高会掩盖低含量，使不合格品看起来合格。
    - **常见错误**：审核时断言"偏高效应不可能导致假阴性"。这是错误的推理。正确表述应为："偏高效应理论上可能导致个别历史批次存在假阴性判定，无法排除已放行不合格品的可能性。"
    - **厂家COA对比作为独立验证的前提**：厂家必须使用**不同原理**的检测方法（如HPLC vs 非水滴定）。如果厂家使用相同方法和相同误差源（如同样的电极污染风险），厂家COA与我司结果的一致性不能作为独立验证证据。此时COA对比仅能定位为"参考性数据"。
    - **合规结论模板**：当无法追溯历史批次检测时的仪器状态，且偏高效应理论上可导致假阴性时，正确的GMP表述是"无法排除个别历史批次存在假阴性判定的可能性，建议在年度质量回顾中增加留样复测以最终确认"。而不是"不会对产品质量产生不良影响"。
    - **自检问题**：在生成D4风险表述评估前，先问自己——"这个偏高效应/偏低效应是否可能导致假阴性？"如果答案是"可能"，则不能下"无影响"的结论。

11. **防御性叙事检测 — 看落点，不看定量论证的量.** 防御性自证的本质不是"定量论证太多"，而是落点错了：同一个 6000 倍过量计算，落点在"不会产生实质影响" = 减责（应删），落点在"即使氧化至检查不可见程度，结果仍可靠" = 评估（应保留）。**删除技术分析前，先判断它是否承担了"隐性偏差识别期"的兜底功能**——即偏差已发生但肉眼/常规检查无法识别的时段。

**判定顺序**：①先看落点主语（缺陷 or 结果）②再看是否承担窗口期兜底功能 ③最后才决定删/留

┌─ 触发条件：缺陷涉及渐进性变化（氧化/降解/吸湿/吸附等）时展开详细检查 ─┐
│                                                                              │
│ 隐性偏差识别期判定标准：                                                       │
│ 1. 试剂/物料是否具有已知的光/热/氧化敏感性？                                    │
│ 2. 储存条件是否符合规定（如避光要求）？                                         │
│ 3. 常规外观检查的灵敏度是否足以识别早期变化？                                   │
│                                                                              │
│ 若三项均为"是"，则存在隐性偏差识别期，需要技术论证补充（详见Pitfall 15）       │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘

12. **New pitfall — Root cause depth trap:** "缺规定" is a phenomenon, not a root cause. Always追问: why didn't the evaluation/assessment mechanism catch this risk? The root cause should point to mechanism failure, not just missing documentation.

13. **整改力度与影响评估结论必须成比例.** 若影响评估结论写得极轻（如只留"使用前检查无异常，结果可靠"一条弱论据），整改力度却极重（换瓶+改规程+稳定性考察+全员培训），这种失衡本身就是审计追问点——审计员会本能地问"如果影响真的那么小，为什么整改这么重？"。正确平衡：影响评估用足证据（技术兜底 + 调查检验），整改力度与真实影响范围成比例。终稿两个方向都可能走错：影响评估偏轻（弱论据支撑"结果可靠"），或整改力度偏重（超出真实影响范围）。

14. **程序优先原则违反 — 结果解释不能替代程序调查.** 在GMP调查中，"程序合规"是"结果可靠"的前提。禁止三种逻辑倒置模式：

- **模式A（结果倒推程序）**：用"结果看起来正常"证明"程序没问题" → 举证责任倒置
- **模式B（假设异常论证结果）**：主动假设程序异常并用计算论证"结果不受影响" → 覆盖调查义务
- **模式C（技术替代调查）**：用技术结论作为调查终点，声称"计算显示没问题，不需要进一步调查" → 掩盖系统性缺陷

自检问题：报告中是否先确认了程序合规性？技术计算是用于"证明程序合规"还是"评估程序异常后的影响"？前者违规，后者合规。

┌─ 触发条件：发现上述模式A/B/C表述时展开详细分析 ─┐
│                                                    │
│ 详细模式定义、案例对照、修正模板见                   │
│ references/pitfall-14-detailed-modes.md            │
│                                                    │
└────────────────────────────────────────────────────┘

15. **方法稳健性论证的来源合法性（条件触发）** — 引用"系统稳健性"或"过量设计"作为技术兜底时，必须先确认方法的来源和验证状态。不能事后补证。

┌─ 触发条件：报告引用"系统稳健性"、"过量设计"、"方法设计考虑了XX波动"时 ─┐
│                                                                          │
│ 药典方法：不得声称"设计了X倍过量"（药典规定≠企业设计）                    │
│ 自建方法：需有验证报告支撑（报告编号+具体章节）                            │
│ 内控方法：需有适用性评估记录                                              │
│                                                                          │
│ 审核追问：来源？验证？变更控制？论证用途？                                 │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘

详细触发指南见 references/pitfall-15-trigger-guide.md

16. **模糊措施动词陷阱 — CAPA 写了≠可验收.** 行动项用方向性动词时，等于没写措施。审核必查：措施动词是否落到可交付动作（修订/更换/增加/执行/销毁），还是停留在"梳理/调查/了解/加强/强化/巩固/完善"等无法验收的模糊词。

┌─ 判定表 ─────────────────────────────────────────────────────┐
│ 模糊（打回）                    │ 可验收（放行）              │
│ 梳理有效期清单                 │ 修订清单，玻璃瓶→棕色蓝盖瓶  │
│ 加强检验员培训                 │ 完成异常峰识别培训并考核 │
│ 完善SOP                       │ 修订SOP增加图谱+干扰峰判定条款│
└──────────────────────────────────────────────────────────────┘

- **每条措施三要素**：具体动词 + 交付物（附件/文件/记录）+ 责任人（具体到人，不得写"QC"）+ 完成时限。缺一即 D6 降分。
- **自检**：把措施句里的动词换成"按附件X执行Y"，若说不通/无法验收，就是模糊动词。
- 案例：异常峰缺陷 V3.0 的 5.2"强化红外分析异常峰识别"——"强化"是 Pitfall 16 点名模糊词，应改为"完成培训并留存考核记录"（见 references/case-study-异常峰缺陷报告.md）。

## Reference Files

- `references/c-style-adaptation-patterns.md` — C式审核三种增强模式（加镜子/加过滤网/加关系图）及各技能适配详情、验证结果
- `references/batch-audit-campaign.md` — 批量审核工作流：对大量历史文档进行统一标准审核的完整流程、分组策略、命令链和输出模板
- `references/pharmacopoeia-comparison-audit-example.md` — 药典对比报告审核实例：10个通则方法的完整审核案例，含D1/D3/D4/D5评分细节和关键发现
- `references/pharmacopoeia-comparison-checklist.md` — 药典对比审核Checklist：完整性、准确性、推论正确性三项审核要求，与8维度C式审核框架的映射关系
- `references/path-update-verification.md` — 文件目录迁移后的路径更新验证清单（Skill引用、Obsidian链接、gmp profile同步）
- `references/audit-to-revision-workflow.md` — 审核→修订工作流：work-summary类型D5权重修正（0.5→1.0）、Mode B修订7步法、文件命名规范（2026-07-19案例）
- `references/case-study-某无机盐缺陷报告.md` — 某无机盐缺陷报告三版迭代案例（2026-08-28）：C式审核六大原则、完整决策链、五级归因链、质量事件通用检查清单
- `references/case-study-包材水分OOS.md` — 包材水分OOS调查报告 B/C 差异分析（2026-07-18，skill最初案例）：5维度差异、数据桥接证据、待改进项4缺口
- `references/case-study-某无机盐缺陷报告-v0.5补充.md` — v0.5审核后应用案例：程序优先原则、方法来源区分、隐性偏差识别期判定、Pitfall 14/15违反模式识别
- `references/case-study-包材水分OOS-v0.5补充.md` — v0.5审核后应用案例：方法来源区分、适用性与稳健性概念辨析、CAPA分层
- `references/case-analysis-v0.5-comparison.md` — 两个案例对比分析：共性模式与差异点、通用审核要点
- `references/pitfall-11-detail.md` — Pitfall 11扩展：隐性偏差识别期判定标准、案例对照
- `references/pitfall-14-detailed-modes.md` — Pitfall 14扩展：三种违规模式详解、自检清单
- `references/pitfall-15-trigger-guide.md` — Pitfall 15扩展：方法来源区分表、审核决策树
- `references/case-study-异常峰缺陷报告.md` — 异常峰缺陷三版迭代案例（2026-09-06）：视角切换时事实锚点须保留（D7第6点）、检查缺陷/缺陷描述二分（D0专属操作法）、历史回顾+复测排除数据桥接（D3）、模糊措施动词（Pitfall 16）

## 审核报告存档

v0.6 版本审核报告已移至独立目录（`<审核报告目录>/review-<主题>-v0.6.md`），便于归档和查阅。

## Verification

Confirm the review was applied by checking:
- All applicable dimensions scored with specific evidence quotes
- Each sub-threshold dimension has at least one concrete rewrite suggestion
- Red-flag items are actionable and time-bounded
- The output preserves what the original author did well
- Weighted score calculation is correct
