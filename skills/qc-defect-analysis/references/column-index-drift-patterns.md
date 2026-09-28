# Column-Index Drift Between Years

Multi-year QC Excel sheets often have different column layouts. Common trap: the same
field label shifts index between years.

## Real Example: QC Invalid Test Data

**2025 sheet** has a dummy "A" prefix column in the header that data rows don't have:

Header: ['A', '月份', '无效测试编号', '类型', ..., '基础原因分类', '详细原因分类', ..., '组']
Data:   ['1', '蒋某某2025.01', 'InA-QC-...', '系统适用性', ..., '人员操作', '人员疏忽', ..., '理化组']

r[0] / col A in data = 月份 (NOT 'A' from header)
r[10] = 基础原因分类, r[11] = 详细原因分类, r[16] = 组

**2026 sheet** has a clean layout with no prefix:

Header: ['月份', '发起人/时间', '无效测试编号', ..., '基础原因分类', '详细原因分类', ..., '组']
r[0] = 月份, r[11] = 基础原因分类, r[12] = 详细原因分类, r[18] = 组

## Detection Method

1. Print header row and first 2 data rows side-by-side
2. Check if header[0] is a column letter ('A','B') while data[0] is meaningful
3. If yes, header has an extra dummy prefix — do not map indices from headers directly
4. Walk field-by-field: "does this data value look like what the header label says?"

## Team Name Evolution

Team names also drift. Known transitions:

| 2025 | 2026 |
|------|------|
| 理化组 | 产品理化 |
| 小分子组 | 小分子 |
| 微生物组 | 微生物 |

Always collect ALL unique "组" values and ask user which to include.
