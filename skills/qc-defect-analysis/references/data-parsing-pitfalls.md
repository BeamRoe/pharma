# 数据解析陷阱（Excel 直解 · 编号 · 表头 · 排序）

> 2026-09-25 由 `oox-subset-deep-analysis` 与 `qc-data-pitfalls` 两条 skill 归并而来（两条已归档）。

## 1. xlsx 打不开：两种症状、两套修法

### 症状 A — GUID 大小写/格式损坏（WPS 类导出常见）

- 报错：`ValueError: Value does not match pattern {[0-9A-F]{8}-...}` 或 `Value must not be none`
- 根因：`xl/workbook.xml` 的 `<customWorkbookView>` / `<definedName>` 中 GUID 为小写
- **修法（只修副本，绝不动原文件）**：zipfile 读全部条目 → 正则**删掉整个 `<customWorkbookViews>...</customWorkbookViews>` 元素**（不是只删 guid 属性，否则产生空字符串同样非法）→ 清理 definedName 里的 GUID 前缀 → 逐条写入新 zip → 重新 load 验证。
- ⚠️ **绝不在同一 with 块里既读又写 zip**，否则触发 `ValueError: Attempt to use ZIP archive that was already closed`。
- 反面案例：修复脚本覆盖了原 zip 但写入失败 → 产生 22 字节空 ZIP（仅 EOCD）。发现文件大小异常时先用 zipfile 检查结构。

### 症状 B — 样式表损坏

- 报错：`expected <class 'openpyxl.styles.fills.Fill'>`（pandas 同样挂）
- **修法（降级直读 XML）**：解包 xlsx → 读 `xl/worksheets/sheetN.xml` + `xl/sharedStrings.xml` → ElementTree 遍历 `//m:c`，`r` 属性解析行列、`t="s"` 走 sharedStrings 下标、`<v>` 直读；rich text 的多段 `t` 节点需拼接；日期列可能是 Excel serial（如 46036）→ `date(1899,12,30)+timedelta(days=serial)`；sharedStrings 索引为空值时用 `if val else ''` 兜底防 `int('')` 崩。**这是可复现的解析路径，不是"读不了"**。

### 通用

修复副本留在分析目录做审计留痕，并在报告数据质量章节写明。完整三源实例（styles.xml 损坏的计算机化系统事件表）见 `qc-data-report-kanban-pipeline` 的「偏差三源数据源的技术坑」。

## 2. 编号提取：位数定死是最高频错误

- 正确：`re.search(r"(\d{7})", raw)`，再按前缀判类型 —— `OOT` in upper(raw) → OOT；`00E`/`OOE` → OOE；其余 → OOS。
- 前缀有 OCR 残迹（`00S`/`O0S`/`OOS` 混用）→ **一律重新拼前缀，不要信任原文**。
- 失效场景：编号列整列为 `0.0`（如 25年-4 sheet 的 OOE 编号）→ 从描述正则提取；仍无则用"日期+序号"构造临时 ID 并在报告中标注。

## 3. 表头行位置与列映射

- 每个 xlsx 的 25年/26年 sheet 列布局都可能不同，**表头可能占 1~2 行**（自检类常为：第 1 行是"填写要求"合并格、第 2 行才是表头、数据从第 3 行起）。
- 解析前先用 `iter_rows` 打印前 2~3 行确认列名与数据起始行，再定列号；**不要沿用上一年的列映射**（实例：25-26 偏差初版脚本误用 26 年映射，把 cause_cat 读到了级别列）。
- 逐 sheet 逐格打印验证：打印每个 sheet 首行数据行的 1..N 列值人工核对，再硬编码列号。
- 解析后做字段合理性 sanity check：责任列应是部门代号（QC）、实验室列应是 是/否 —— 出现"是/否"串到责任列即列号错位，回查。
- 部分源的有效记录要按编号特征判定而非按 `max_row`（实例：验证偏差 26 年 sheet 81 行中仅 2 条有效，其余是模板空行）。

## 4. 错位行 / 乱码行

- 合并单元格溢出的行（编号+日期+编号挤在一格、数据整体左移）正则救不回来 → 直接手工硬编码该行字段值（从打印的原始值转录），JSON 里保留 sheet 出处。
- 25 年 sheet 常见：房间列与描述列混在同一个单元格。

## 5. 去重与中间产物

- 跨 sheet 去重：同一事件在多 sheet 出现（`25年-1` 与 `25年-2` 都有 OOS2507010）→ 保留首次出现，记 `dup_sheets`。
- 输出中间 JSON（事件全字段 + sheet 出处），后续所有统计从 JSON 生成，保证可复现、可审计。

## 6. 月份排序

- 月份字段是 `1月`…`12月` 字符串时，排序必须数字化（`int(m.rstrip('月'))`），否则 `10月` 会按字典序排到 `1月`/`2月` 之后。
