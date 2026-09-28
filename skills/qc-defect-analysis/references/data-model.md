# 质量事件主表数据模型

## 主表字段（偏差/OOS/异常测试合并主表）

| 字段 | 示例 | 作用 |
|------|------|------|
| record_id | DEV-2025-00123 | 唯一编号，防止重复统计 |
| record_type | 偏差/OOS/OOT/实验室异常/趋势异常 | 事件类型 |
| site | A工厂 | 厂区 |
| department | 生产/QC/QA/仓储/工程 | 责任或发生部门 |
| product_name / batch_no / dosage_form | 产品A / B20250101 / 片剂 | 产品维度 |
| process_stage | 称量/配液/灌装/压片/包装/检验 | 工艺或活动阶段 |
| test_item | 含量/杂质/微生物/水分/溶出/pH | 检测项目 |
| sample_type | 原料/中间体/成品/稳定性/环境监测 | 样品类型 |
| instrument_id / equipment_id | HPLC-001 / MIX-001 | 仪器/设备 |
| severity | minor/major/critical | 严重性（不允许AI修改） |
| occurrence_date / discovery_date / close_date | 2025-03-15 | 三个日期都保留 |
| root_cause_category / root_cause_subcategory | 人员/设备… | 根因大类/小类（见字典） |
| immediate_cause / final_root_cause | 操作失误 / 已批准根因 | 直接原因 / 受控结论 |
| capa_required / capa_id / capa_status | 是 / CAPA-2025-008 / 已关闭、逾期、进行中 | CAPA关联 |
| recurrence | 首次/重复/相似重复 | 重复性 |
| impact_assessment | 对质量无影响/潜在影响/有影响 | 影响评估 |
| report_status | 已关闭/进行中/作废 | 纳入统计前提 |

## 双层字段规则
原始字段与清洗字段必须分开，清洗规则可追溯：
- 原始：raw_root_cause、raw_department、raw_process_stage、raw_description
- 清洗：std_root_cause_category、std_department、std_process_stage、std_event_type
- AI建议：ai_suggested_category（人工复核后才进 std_*）
审计时可回答：原始记录是什么、标准化规则是什么、最终统计用了什么。

## 根因分类字典（先固定再统计）
一级根因（10类）：
1. 人员  2. 设备/设施  3. 物料  4. 方法/程序  5. 环境
6. 检测/分析方法  7. 文件/记录  8. 管理系统  9. 供应商  10. 未确认根因

二级示例：
- 人员：培训不足、操作失误、经验不足、交接班不足、人员配置不足
- 设备/设施：设备故障、维护不足、备件失效、设计缺陷、清洁不充分
- 方法/程序：SOP不清晰、工艺参数控制不足、检验方法不稳健、取样程序缺陷
- 管理系统：CAPA无效、变更评估不足、趋势回顾不足、供应商管理不足

自由文本根因（"操作失误/未按SOP执行/人为错误/记录填写错误"等）直接统计会碎片化，必须映射到统一分类。映射采用"AI初分+人工复核+锁定分类"。

## 数据源优先级
第一优先级（正式回顾应尽量使用）：eQMS已批准/已关闭的偏差、OOS、CAPA记录；LIMS正式检验结果；MES/EBR正式批生产记录；已批准APQR/PQR数据集
第二优先级（补充）：部门年度Excel台账、人工维护趋势表、会议纪要问题清单
第三优先级（不作统计源）：邮件、聊天记录、个人笔记

## 唯一ID与关联键
- 唯一键必须是系统记录号（DEV-2025-001 / OOS-2025-023 / LAB-INV-2025-015 / CAPA-2025-009），禁止用"产品+日期+描述"
- 一个偏差关联一个OOS和一个CAPA时保留关联关系：deviation_id | oos_id | capa_id，否则重复统计

## 归一化指标（周期对比必配）
- 偏差率 = 偏差数 / 生产批次数
- OOS率 = OOS数 / 检验批次数
- 实验室异常率 = 实验室异常数 / 检验总次数
产量变化时绝对数会误导，趋势结论必须带归一化口径。
