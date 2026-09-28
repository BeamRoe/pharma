# CAPA Quality Assessment Methodology

A critical dimension often missed in QC data analysis: not just HOW MANY defects or invalid tests, but whether root cause investigation leads to actual corrective action.

## The Ratio

CAPA coverage rate = records with actionable CAPA / total records

## Classification Rules

**Actionable CAPA** when capa is non-empty, not literally 'N/A', 'None', or '无'.

**No CAPA** when empty, blank, 'N/A', or '无'.

## Meaningful vs Non-Meaningful Examples

| Meaningful (systemic fix) | Non-meaningful (no action) |
|---------------------------|----------------------------|
| "修订SOP《xxx》，明确操作要求" | "N/A" |
| "在组内进行复盘培训" | (empty) |
| "仪器报修" | "认为是偶然异常" |
| "更换色谱柱，报废" | "已解释，能接受" |
| "增加排查频率至每半月" | — |

## Red Flags

1. Same person 3+ times with no CAPA = individual coaching need
2. Same instrument repeatedly failing with no CAPA = preventive maintenance gap
3. "偶然异常" catch-all with no CAPA = superficial investigation
4. Defect type systematically lacking CAPA = process gap

## CAPA Effectiveness

| Type | Example | Effect |
|------|---------|:-------:|
| SOP修订 | "修订操作规程" | High |
| 培训复盘 | "组内复盘培训" | Medium |
| 仪器维修 | "仪器报修" | Medium |
| 耗材更换 | "报废色谱柱" | Low-Med |
| 频率调整 | "增加排查至每半月" | High |
| N/A空白 | — | None |

## Benchmarks

| Coverage | Assessment |
|:--------:|:-----------|
| >60% | Good |
| 30-60% | Fair |
| <30% | Critical |

Any ratio below 30% is a critical finding regardless of defect count reduction.
