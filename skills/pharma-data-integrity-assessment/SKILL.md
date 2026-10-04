---
name: pharma-data-integrity-assessment
description: 基于「系统稳健性」模型，定量评估一起质量事件（试剂变色、仪器漂移、环境偏离、人员异常、方法缺陷）发生之前的历批数据是否仍然可信，用控制图、置换检验、效应量检验历史数据有无异常，输出可靠/有条件可靠/不可靠/无法判定四级结论与波及范围建议。当用户要求「这批试剂变色了，之前的数据还能用吗」「历史数据还能不能采信」「要不要复测留样」「往前追溯多少批」时触发。
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags:
    - 数据完整性
    - 历史数据
    - 统计
    - 控制图
    - 置换检验
    - 追溯
    related_skills:
    - pharma-qe-s3-risk-assessment
    - pharma-gmp-gap-scan
---

# 基于系统稳健性的历史数据可靠性评估

> 定位：**这是全套技能里唯一必须靠计算说话的一个。**
> 传统的做法要么"全部复测"（成本高、且复测本身也不一定更有说服力），要么"拍脑袋说没事"（审计必被挑）。
> 本技能给出第三条路：**用系统能不能自证 + 数据有没有异动，来科学界定追溯范围。**

## 铁律

1. **统计计算一律走脚本，禁止 LLM 心算或"估算"。** 所有数值必须由 `scripts/systemic_robustness.py` 产出。
2. **样本不足不给结论。** n < 10 或事件前 n < 3 → 直接返回 `indeterminate`。宁可说"不知道"。
3. **先问"能不能抓住"，再问"有没有异常"。** 一个连系统适用性都没有的体系，历史数据再漂亮也无法自证——这是逻辑顺序，不能颠倒。
4. **不给放行结论。** 输出的是"数据可靠性证据"，不是"可以放行"。

## 核心模型：稳健性三支柱

```
        稳健性 R（0–100）
        ┌───────────┼───────────┐
        │           │           │
   D 检测能力    S 统计信号   C 补偿性控制
    (0–40)       (0–30)       (0–30)
        │           │           │
  事件发生时有   历史数据本身   除检测手段外
  没有当场抓住   有没有漂移/    还有几层独立
  它的手段？     失控的痕迹？   的兜底？
```

**D 检测能力（0–40）** —— 最关键的一问
| 项 | 分值 | 含义 |
|---|---|---|
| 系统适用性检查 | 12 | 每批/每次分析都有 SST、对照品、阳性对照等"自带标尺" |
| 独立复核/第二方法 | 12 | 有第二人复核或独立方法交叉验证 |
| 趋势回顾 | 8 | 有定期趋势分析（月度/年度回顾） |
| 时间窗可界定 | 8 | 能精确界定事件起止（否则无法定范围） |

> 一句话解释给审计员：**"如果系统连'抓住问题的能力'都没有，那么'没发现问题'就不能作为'没有问题'的证据。"**
> 这就是绝大多数"历史数据无需追溯"结论被挑战的根因。

**S 统计信号（0–30）** —— 数据有没有自己"发声"
- Western Electric 判异（1点超3σ / 连续9点同侧 / 连续6点趋势 / 连续14点交替）
- 事件前后均值差的**置换检验**（固定种子，可复现，不依赖正态假设）
- Cliff's delta 效应量（|δ|<0.147 视为可忽略）
- Cpk/Ppk 过程能力

**C 补偿性控制（0–30）** —— 层层兜底
每项独立控制 +10，非独立（同一人/同一环节）+5，上限 30。

## 结论分级矩阵

| R 分 | 无异常信号 | 有异常信号 |
|---|---|---|
| **≥75（高）** | ✅ reliable 可靠 | ⚠️ reliable_with_conditions 需针对性确认 |
| **50–74（中）** | ⚠️ reliable_with_conditions 需补一项独立确认 | ❌ not_reliable 需逐批评估 |
| **<50（低）** | ❌ not_reliable 数据无法自证，需独立评估 | ❌ not_reliable |
| **样本不足** | ⛔ indeterminate 不做推断 | ⛔ indeterminate |

## 执行流程

### 步骤 1 · 收集并结构化数据
把历史检验结果整理成 `case.json`（见输入契约）。**不要**在对话里直接贴几十个数字让模型算。

### 步骤 2 · 运行计算引擎
```bash
python3 <skill 目录>/scripts/systemic_robustness.py --input case.json --out result.json
```
返回码：`0` = 成功；`2` = blocked（数据不足或参数非法）。
**脚本返回 2 时，禁止自行编造结论，必须向用户说明阻塞原因。**

> **脚本路径（本机）**：脚本相对本 skill 目录：`scripts/systemic_robustness.py`；示例数据：`examples/case_reagent_discoloration.json`。
> 若目标 harness 以项目根为 cwd，需先定位本 skill 目录再拼接 `scripts/`；若提供 `${SKILL_DIR}` 之类的变量则优先使用。
> **校验基准**：用 `examples/case_reagent_discoloration.json` 跑出的稳健性得分应为 **71.0**，结论为 `reliable_with_conditions`。对不上即为路径或运行时问题。

### 步骤 3 · 解释数值（这一步才是 LLM 的活）
拿到 `result.json` 后，把统计结果翻译成质量语言：
- 说明 R 分各支柱的贡献与短板
- 说明失控点/异常信号对应哪些批次
- 说明 p 值与效应量的**实际意义**（不是复述数字）
- 若结论为 `not_reliable`，给出追溯范围的边界逻辑

### 步骤 4 · 输出结论 + 追溯建议

## 输入契约（case.json）

```json
{
  "series": [
    {"batch": "B123456", "date": "2026-01-05", "value": 99.2},
    {"batch": "B123456", "date": "2026-01-12", "value": 98.7}
  ],
  "event_index": 12,
  "spec": { "lsl": 95.0, "usl": 105.0, "target": 100.0 },
  "detection": {
    "system_suitability": true,
    "independent_check": false,
    "independent_check_partial": true,
    "trending_review": true,
    "time_window_bounded": false
  },
  "compensating_controls": [
    {"name": "QC 双人复核", "independent": true},
    {"name": "月度趋势回顾", "independent": false}
  ]
}
```

| 字段 | 必填 | 说明 | 缺失策略 |
|---|---|---|---|
| series | ✅ | 按时间顺序排列的批次与检测值 | 拒绝 |
| event_index | ✅ | 事件首次显现的批次序号（0-based） | 拒绝（这是范围边界的锚点） |
| spec.lsl / usl | 选填 | 规格限，用于 Cpk | 跳过能力分析 |
| detection.* | ✅ | 四项检测能力，布尔；`_partial` 后缀表示部分具备 | 视为 false |
| compensating_controls | 选填 | 补偿性控制清单，标注是否独立 | 计 0 分 |

## 输出契约

脚本输出结构化 JSON（`status / sample / descriptive / capability / control_chart / shift_test / robustness / conclusion`），
技能在其之上补充**人类可读的解释层**：

```yaml
result:
  skill: pharma-data-integrity-assessment
  version: "1.0.0"
  status: ok
  engine_output_ref: <result.json 路径>
  robustness_score: 62.0
  dimension_readout:
    - "D 检测能力 20/40：具备系统适用性与趋势回顾，但缺少独立复核、事件时间窗无法精确界定"
    - "S 统计信号 30/30：控制图无判异，事件前后均值差 p=0.41，效应量可忽略"
    - "C 补偿性控制 12/30：仅 1 项独立控制"
  conclusion:
    level: reliable_with_conditions
    plain_language: "系统能在一定程度上自证，且历史数据未显示异常；但独立复核缺失，建议对事件时间窗内批次加做一次留样复测以补强证据。"
  affected_scope:
    suggested: "事件时间窗内批次（B123456–B123456）"
    boundary_logic: "以试剂启用日为起点、以变色发现日为终点"
    confirmatory_actions:
      - "抽取该区间首、中、末共 3 批留样复测"
      - "复测方法需与原始方法一致或经确认等效"
  gaps: [ "事件时间窗起点依赖人员回忆，建议核对试剂领用记录" ]
  confidence_overall: medium
  human_review_required: true
  disclaimer: "本评估为技术性证据，不替代质量负责人的数据处置与放行决定。"
```

## 治理层

- **可复现即可审计**：脚本固定随机种子（SEED=20260101），同一份输入任何人运行都得同一结果。这是应对"你的结论怎么来的"最有力的回答。
- **纯标准库、无网络、无外部依赖**：便于在受控/验证环境中部署，也便于做 CSV（计算机化系统验证）确认。
- **失败即报**：数据不足返回 `blocked`，**绝不**降级输出一个"看起来像结论"的判断。
- **ALCOA+ 对齐**：输出的 `input_echo` + `generated_at` + `engine_version` + `reproducibility` 字段，构成完整的审计追踪要素。
- **不越权**：本技能回答"数据可不可信"，**不**回答"产品能不能放行"。
