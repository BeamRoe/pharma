# HTML报告输出模式 — qc-defect-analysis

## 背景

每次缺陷分析都需要生成交互式HTML报告。为避免重复生成大型HTML字符串导致 `write_file` 超时（>8K token payload），采用分块写入策略。

## 分块写入策略

### Step 1: 创建基础HTML骨架
```
write_file(content='<!DOCTYPE html>...<style>...</style></head><body><div class="container"><div class="header">...</div>')
```
包含：DOCTYPE、head（CSS+ECharts CDN）、header、修订历史、第一节内容。

### Step 2: 追加各章节
```
write_file(content='<div class="section"><h2>二、...</h2>...')
```
每次追加一个完整 `<div class="section">...</div>`，不超过4KB。

### Step 3: 追加图表脚本
```
write_file(content='<div class="section"><h2>四、图表</h2>...<script>echarts初始化...</script></div>')
```
图表用 ECharts（CDN引入），每个图表 <200行JS。

### Step 4: 组装最终文件
```python
parts = []
for i in range(1, N+1):
    parts.append(read_file(f'part{i}.html'))
final = parts[0] + ''.join(parts[1:]) + '</div></body></html>'
write_file(final, path='final_output.html')
```

### Step 5: 清理临时文件
```python
for i in range(1, N+1):
    os.remove(f'part{i}.html')
```

## CSS类约定

| 类名 | 用途 |
|------|------|
| `.section` | 章节卡片（白底、阴影） |
| `.section h2` | 章节标题（左侧蓝色边框） |
| `.section h3` | 小节标题 |
| `.chart-row` | 图表行（flex布局，两个图表并排） |
| `.chart-box` | 单个图表容器 |
| `.chart-container` | ECharts div，height:350px |
| `.summary-box` | 蓝色左侧边框的信息框 |
| `.warning-box` | 橙色左侧边框的警告框 |
| `.detail-grid` | 2列网格布局 |
| `.detail-card` | 卡片内嵌信息 |
| `.highlight` | 红色加粗（异常/高亮数据） |
| `.good` | 绿色加粗（正面数据） |
| `.dt` | 数据表格 |

## ECharts图表类型库

| ID | 类型 | 用途 |
|----|------|------|
| ch1 | pie | 缺陷分类饼图 |
| ch2 | bar (stacked) | 年度对比柱状图 |
| ch3 | line | 月度趋势折线图 |
| ch4 | bar (horizontal) | 全组vs子组对标 |
| ch5 | radar | 多维能力/风险雷达图 |
| ch6 | bar (labeled) | 评分/排名柱状图 |

## 注意事项

- 每个 `write_file` 调用限制约 8K tokens payload，超过会超时
- 临时文件命名：`文件名_V1.0_part{N}.html`
- 最终输出命名：`文件名_V1.0.html`
- 图表容器用 `resize` 事件监听确保响应式
- ECharts CDN: `https://cdn.jsdelivr.net/npm/echarts@5/dist/echarts.min.js`

## 另一种模式：单文件自包含 HTML（无 ECharts，轻量交付）

用户直接要求"按 html 格式生成分析报告"（不走 PPT/kanban）时用这种：内联 CSS、**不引 ECharts**、纯表格 + KPI 卡片 + callout + CSS bar，浏览器直接打开即可；存到数据源同目录，命名 `{主题}_{日期}_V1.0.html`。

报告内固定设「完整性核验」节（遗漏检查 / 编造检查 / 遗留待核项），产出后用 Python `html.parser` 校验标签闭合再交付。完整规范与上文交付流程见 `qc-data-report-kanban-pipeline` 第 3.5 节与「HTML 产物直发 gmp 审核」节。
