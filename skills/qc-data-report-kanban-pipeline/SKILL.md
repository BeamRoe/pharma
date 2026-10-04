---
name: qc-data-report-kanban-pipeline
description: Use when QC 分析报告需经 kanban 交 gmp 审核定稿后再交 ppt 制作。
version: 1.2.0
author: oos profile curation 2026-09-15; 命令语法复验 2026-09-29
metadata.hermes:
  tags: [kanban, gmp-review, ppt, quality-report, pipeline, multi-profile]
  related_skills: [qc-defect-analysis, oos-investigation-review, powerpoint]
---

# QC 质量报告 Kanban 交付流水线（分析 → GMP 审核 → PPT）

适用场景：用户要求分析 QC 质量事件数据（自检缺陷、OOX 频率趋势等），出分析报告，然后「通过 kanban 交 gmp 审核，定稿后交 ppt 制作 PPT」。本 skill 沉淀 2026-09-15 两次完整交付（OOX 报告 + 物料组自检缺陷报告）的流程经验。数据分析本体走 `qc-defect-analysis`，报告 C 式质量走 `oos-investigation-review`；本 skill 只管交付回路。

## 交付回路分两类（别把 PPT/HTML 强加给所有 gmp 审核任务）

| 类型 | 下游 | PPT/HTML | 说明 |
|---|---|---|---|
| **数据/回顾报告**（qc-defect-analysis 产出：OOX 频率趋势、缺陷台账、无效测试分析） | gmp 审核 → 定稿 → **ppt worker** | ✅ 走 PPT（`--parent` 挂审核卡） | 本 skill 主线 |
| **单独质量事件调查报告**（pharma-qe 主链 S2b 产出：OOS/OOE/OOT/偏差/无效测试单事件） | gmp 审核（**AI 终审 = 人工审核前一步**）→ **人工签** | ❌ **不建 PPT、不生成 HTML** | 审核体裁 = `oos-investigation-review` C 式 8 维审**调查报告**（非数据报告审核）；gmp 复审定位 = 人工审核（质量负责人 D1 根因终点/D2 定级/D3 CAPA/D4 上报）之前的 AI 终审 |

**单独事件报告走 gmp 的三个特征**：① 审核意见只出**判定层**（红/黄项清单 + 关键判定，见「单独事件调查报告走 gmp 的 body 重量」节），不逐条引原文改写；② 审核通过后**没有 ppt worker 接手**——下一步是人工审核（质量负责人终裁），不是 PPT；③ 单独事件报告定稿后**不生成 HTML**（HTML 是数据回顾报告的可选轻量产出，单事件报告不做）。任务卡 body 里要显式写「单独事件报告，不建 PPT 任务、不生成 HTML」，否则 gmp 可能默认套数据报告流程。

> **单独事件的 S6 = 两级（2026-09-27 用户明确）**：① **AI 终审** = kanban 派 gmp 跑 `oos-investigation-review`，产出固定命名 `03d_gmp_review.md`（红/黄项 + 关键判定）；② **人工最终批准** = 质量负责人 / 审核人对 D1 根因终点 / D2 定级 / D3 CAPA / D4 上报签字。gmp 出意见后：oos 核对意见引用事实相符性 → 在副本上修订 → gmp 轻量复审 → 人工签。**gmp 崩溃 / 离线时挂起报原因，不得在 oos 内自审收口**（实测偏离：2026-09-26 某表面活性剂一案 S6 首轮在 oos 自审）。
> 任务卡 body 直接套 `templates/gmp_audit_body.md`，输出文件名写死 `03d_gmp_review.md`（不要用 `gmp_audit_*.md` 别名）。

## 根因深度 L4/L5 + 审核建议与既定结论的衔接（2026-09-26 单事件调查报告）

> 单独事件调查报告根因常停在 L3/L4（"规定内容缺口"）。gmp 可能建议上溯 L5（管理根因层，如"变更程序为何没把 X 列为必查项"）。
> **gmp 建议是审核意见、非定论**——根因终点是**人工终裁项**。oos 修订时：① 若用户既定 L4，**维持 L4 但按 gmp 要求补声明**（"若维持 L4 须声明 L5 未评估的假设/证据，或注明 L5 管理根因留待专项评估"）；② **不改判 L5**（改判=结论层改动，须退 S2b 出新版重走 S5，不在 S6 改）；③ 用户若有更深洞察（如本案"缺的是对新厂家的质量全面评价"，比 L4/L5 都准）→ 把根因**定性升级**（维持落点层、升级缺口性质），同步升级 CAPA 对应措施（本案 P1 从"强制单杂谱比对"升为"变更后做质量全面评价"），根因与措施才自洽。
> 判据调用记入 `_meta/judgment_trace.yaml`（J1 终点 + pending_reviewer_decision），留 S6/人工终裁。

## 任务图（两步 + 依赖链接）

```
分析+报告（oos 本 profile 做）
   ↓ kanban create → gmp profile
GMP 审核（gmp worker：逐条核对数据/口径/措施，comment 出 R/Y 意见，需修订则 block）
   ↓ oos 按意见在副本上修订（V1.0→V2.0，末尾加「修订历史」段落）
   ↓ kanban comment 附逐条处理说明 + kanban unblock
GMP 复审 → done
   ↓ PPT 任务（--parent 挂在审核任务上）自动 ready
PPT 制作（ppt worker：基于定稿报告出 pptx，存同目录）
```

## 创建任务的操作细节

1. 任务 body 必须自包含（worker 无会话上下文）：报告绝对路径 + 数据源绝对路径 + 逐条核对要求清单（总量/占比表/归并口径/风险定级/措施优先级）+ 产出物要求清单（如「逐条列出判定依据+去向核验表」）。审核任务的 body 就是 gmp 的验收标准，写多细审核就多细。
2. PPT 任务用 `--parent <审核任务ID>` 创建：父任务 done 时自动 ready；父任务 blocked 时保持 todo——**blocked 是正常待修订状态，不是故障**。PPT body 注明「基于定稿报告（GMP 定稿后版本更新）」，防止 worker 基于 V1.0 旧版制作。
3. PPT 要求写明：覆盖章节、重点图表清单（柱状/折线/饼/条形各对应哪张表）、风格约束（GMP 质量回顾汇报，每页要点≤6 条）、输出格式与保存目录。
3.5. 用户直接要求「按 html 文件格式生成分析报告」（非走 PPT/kanban 的轻量产出）：直接产出单个自包含 HTML（内联 CSS、无 ECharts 依赖、纯表格+callout 卡片），存到数据源同目录，命名 {主题}_V1.0.html；数据源声明要写明 Excel 为唯一事实源、CSV 快照（日期+缺件说明）仅作对照；报告内附「完整性核验」节（遗漏检查：全库 N=子集 m+非子集 N-m 逐条列明去向；编造检查：每个数字/批号可脚本复现、逐条引源表原文；遗留待核项：源表疑似录入错误按源表执行+留痕不擅自修正）——这是用户对「是否有编造/是否遗漏」的标准回应格式。产出后用 Python HTMLParser 校验标签闭合再交付。
4. 查看进度：`hermes kanban ls`（过滤 running/todo）+ `hermes kanban show <id>`（Events 尾部连续 heartbeat = worker 活跃；reclaim/crashed 事件后跟 claim+spawn 是正常回收重派机制，非故障）。验证产出物进度直接查目标目录（deck.html 是否生成、slide 数、assets 是否齐全），比等 worker 写 summary 快。同一 gmp worker 被两个审核任务占用时串行排队，第二个任务 running 时间显著拉长属正常，勿误判卡死。
5. 用户问「任务返回后会自动执行吗」的标准答案：审核任务 **done** 时子任务 PPT 自动 ready 并被 ppt profile 认领；审核任务 **blocked**（需修订）时 PPT 保持 todo，等人工修订+unblock 后复审。**修订环节必须人工做（oos profile），自动化的边界止于审核任务达到 done。**

## GMP 审核意见的复核闭环（最高优先级规则）

gmp 审核意见不是无条件执行的指令，遵循用户既定 GMP 审核规则：

1. **先核对事实**：意见引用的事实/数据/条款是否与源数据（Excel 原件、CSV 对齐数据、报告原文）相符，相符才按意见修订；不符则判为审核意见自身缺陷，不执行，在意见载体（kanban comment）回写「不采纳理由」并附证据。
   - 实例（2026-09-15）：gmp 红项①「25年应为 OOS×11/OOT×0」——逐条清点 11 件 OOS 编号后确认意见正确，照改。红项②「Excel 为 2026.02.09 有误」——重新读 Excel 原件发现 26年 sheet 关闭日期列实际值就是 2025.02.09（与 CSV 相同，源文件录入错误），意见结论对（应为 2026.02.09）但论据错，修订时按正确结论改、写明推断依据，不照抄意见的错误论据。
2. **定性模糊项不擅自改判**：根因分类归并存疑时（如「手阀未关」被标「器具/材料」）标「存疑，待 GMP 终裁」，沿用「定性模糊先与检查人确认」的既有偏好。
3. **修订在副本上**：V1.0 保留，新建 V2.0（命名：{报告名}_V2.0.md），末尾加「修订历史」段落（保留近三版）；过程版等用户告知「批准」后才统一清理。
4. **unblock 前必须 comment**：逐条写「红项X→已改/黄项X→维持存疑的理由 + 新版路径」，审核方复审的是新版不是旧版。

### S6 闭环：审核卡完成后如何建复审卡（2026-09-28 D26051 实测）

**关键认知**：gmp 审核卡出 03d 意见后状态即 `completed`（不是 blocked）——**不能 unblock completed 卡**。oos 按意见修订完副本后，轻量复审必须**新建一张卡**挂 `--parent` 到原审核卡：

1. **复审卡 body 模板**：套 `templates/gmp_s6_recheck_body.md`（S6 复审专用，审修订后报告本体——与 `gmp_lightcheck_body.md` 的 S1/S5 **闸门**抽检区分开），写明：审核对象路径（修订后 V(n+1)）+ 定向复核范围（逐条红/黄项编号 + "无需全量重核"）+ 附带核对（版本引用一致 + 结论未误改）+ 判定标准（N 处全到位且结论未变 → complete；任一未到位 → block 回 oos 补）。
2. **原审核卡 comment**：修订完成后在**原审核卡**上 `hermes kanban comment <原卡id> "oos 已按 03d 意见修订（V2.0→V3.0），逐条处理说明：R-1→已改…Y-1→已改…"`——留审计痕迹，标明复审卡是这张卡的闭环。
3. **新建复审卡**：`hermes kanban create "GMP轻量复审: 03 V(n+1) 定向复核" --assignee gmp --parent <原审核卡id> --body "$(cat /tmp/gmp_recheck_body.txt)"`。body 写进临时文件再命令替换（保证换行真实，同「任务 body 重量控制」节要求）。
4. **限流处理**：gmp run 命中 `rate_limited`（exit_code 75）后卡自动转 `ready` 由 dispatcher 重新 claim（下一 run 编号），**非故障、不干预**；反复 rate_limited 才需拆薄 body。判别细节（Events 里 `rate_limited` 后是否跟新 `claimed`/`spawned`）见 `references/kanban-command-cheatsheet.md`「限流重试」。
5. **复审通过 → 人工终裁**：gmp 复审 complete 后，S6 进入 ②人工最终批准（质量负责人签 D1-D4，`07_final_review.md`）；gmp 复审 block 则 oos 补修后再次建复审卡。

## 根因分类字段的源数据口径（多源数据专用）

1. **优先级**：Excel 原始 sheet 的分类列 > 对齐数据 CSV > 按描述补派。有分类列的年份必须以其原始标注为准；无分类列的年份取 CSV，CSV 缺失的按描述补派并显式标注〔派生〕。
2. **双维计数**：标注「机、法」类双维值按双维各计 1，表格需注明，否则合计对不上（V1.0 曾犯 26年合计=10≠9 的统计错误，出表后必须逐列加总核对）。
3. **日期合法性校验**：关闭日期 < 报告日期 = 必为录入错误，用 Ia/Ib/II/III 阶段日期链推定正确值，报告「数据说明」节写明推断依据。
4. **差异说明要双向核对**：CSV 与 Excel 不一致的每个字段，两个源都要重新读原件确认，不得凭记忆写差异说明（V1.0 曾把「Excel 为 2026.02.09」写错，实际 Excel 也是 2025.02.09）。

## 数据完整性防线（防编造/防遗漏）

- 报告每个数字可由脚本从源文件解析复现；解析+统计代码落盘（/tmp 或工作目录），报告「数据说明与局限」节列出脚本路径与解析口径——这是「是否有编造」的正式回应方式。
- 逐条清单入报告附录（编号/月份/分类/责任人/摘要），供审核人独立核对，回应「是否遗漏」。
- 指定人员过滤：责任人字段混排（逗号多人、换行+角色前缀「填写人：X\n审核人：Y」），用 `any(p in resp for p in PERSONS)` 包含匹配，勿精确匹配；过滤后打印被剔除行（SKIP 日志）供核对。
- 非缺陷记录（「是否定义为缺陷=否」的建议项）单独计数说明，不混入缺陷统计，但提示属跟踪范围。
- 跨年对比口径：期间不等（25年1-11月 vs 26年1-8月）时总量折算月均并注明覆盖月份；排序变化比绝对量更可靠。
- 用户说明「部分不完整记录是暂未完成调查」时，在修订历史备注中记录此口径，不完整字段不编造补全。

## openpyxl 读不了的 xlsx 先修再析

「customWorkbookViews GUID 异常」（`ValueError: Value does not match pattern ...`）：用 zipfile 提取全部条目 → 正则删除 workbook.xml 中整个 `<customWorkbookViews>` 元素 → 清理 definedName 中 GUID 前缀 → 逐条写新 zip 到 /tmp → 重新 load 验证。绝不在同一 with 块中既读又写。详见 `qc-defect-analysis` 的已知陷阱（本 skill 不重复脚本全文）。

## 数据分析类报告的「分母一致性」审核防线（红项高发区）

针对无效测试台账/OOX 子集分析类报告（含空记录、未填写行、未闭环调查行）的高频系统性错误——GMP 审核 2026-09-16 双任务（OOX 物料相关 t_de57133e、无效测试台账 t_49c74cf6）均因分母错误出红项：

1. **剔除行只在分母剔除、未在所有占比分母同步剔除**：无效测试报告把 8 月空记录 260825-02 从「有效 n」中剔除（n=15），却又把它计入「无措施项 8 件」分母 → 全链路占比系统性失真（偶然异常 33.3%→31.25%、无措施 53.3%→43.75%、月均 1.88→2.00）。**规则：报告一旦定义「有效样本 n」，正文所有占比/月均/判定表的分母必须统一用 n；被剔除行（空记录/未填写）只能单列提示，不进任何分母。**
2. **口径含糊的匹配结论必须标注局限**：跨年重复「同项目+检项+原因=0 件」在 26 年 13/16 行项目代码=N/A 的情况下无验证基础（仅 3 行可匹配、结论碰巧对）→ 黄项。写法：结论后括号注明「可匹配范围内」+ 口径局限脚注 + 建议强制填写缺失字段。
3. **出表后自验分母**：表格加总核对之外，逐列检查「分子/分母是否与正文样本口径一致」，空记录/未填写行单独列表、不入分母（与既有「双维计数/日期合法性校验」并列为出表自验三件套）。

## HTML 产物直发 gmp 审核（单卡，无修订回路）

V1.0 HTML 深度分析报告（`qc-defect-analysis` 子集/深度分析模式的产物）走**单卡直发**：`hermes kanban create "GMP审核: <报告名> V1.0" --assignee gmp --workspace scratch`，body 自包含四段：① 审核对象绝对路径；② 背景（数据来源：xlsx 唯一事实源 + 中间 JSON + 交叉验证报告路径）；③ 逐条重点核验清单（判定口径/占比数字/〔派生〕标注/折算口径/遗留待核项——审核任务 body 即 gmp 验收标准，写多细审多细）；④ 产出物要求（审核意见 md 绝对路径 + 完成后 summary 须含红/黄项数量）。

gmp done 后按意见分级闭环（2026-09-16 实测）：
- **0 红 / N 黄（纯文字一致性，如取整口径、前后章节优先级标注不一致）**→ 无需建修订子卡：直接在报告副本上改 V1.1，新建第二张 gmp 卡做轻量复审（body 注明「仅需复核 N 处文字，无需全量重核」）。
- **V1.1 副本修订手法（2026-09-16 无效测试+OOX 物料双任务沉淀）**：V1.0 原文件保留不动；先 patch 源文件落实逐处黄项修订，再 `shutil.copy` 生成 V1.1 副本并替换副本内的 title/版本头/footer（改版本号+修订说明）；grep 残留旧措辞（如「与 P1」「均值 19 天」）=0 为修订到位判据；黄项修订内容同步进 footer 修订说明（哪条黄项→改了什么），供复审 worker 定向核对。黄项若分布在正文与表格两处（如均值口径同时在 §1 趋势表与 §4 明细脚注），两处一并改，避免复审再出同型黄项。
- **有红项**→ 走「GMP 审核意见的复核闭环」既有规则（先核对意见引用的事实与源文件是否相符再修订），建修订卡 + 复审卡。
- 单卡直发的卡也可带 `--parent <关联任务id>` 挂依赖链（如 html-ppt 更新任务 done 后才审报告），不影响上述闭环。

## 任务 body 重量控制（gmp 崩 4 次教训，2026-09-26 单事件调查报告实测）

任务 body 写多细，gmp 单次要产出的审核意见就多重。**调查报告 + 7 条全量核对 + 每项要「引原文+改写建议」+ 多个关键判定 → gmp 写 `03d_gmp_review.md` 时打满模型 max output token，反复崩（单卡 4 crash + blocked，failure_limit 触发）**。

**处置规则**：
1. **gmp 阶段只出「判定层」**：红项/黄项清单（每条一句话：哪节 + 什么问题）+ 关键判定（如根因 L 层、历史数据回溯结论、是否退 oos 修订）+ summary。**逐项改写建议留给 oos 修订阶段自取**，不进 gmp 意见文档。
2. **意见文档限定字数**：任务卡 body 明确写「意见文档 ≤2000 字、只出判定层、不逐项引原文改写」，gmp 单次输出量变小不再撞上限。
3. **崩了先查 `03d` 是否落盘**：`ls` 事件目录看 gmp 崩前有没有写出部分审核意见。崩前未落盘 → 不是「写到一半」，是卡在「读报告 + 生成意见」阶段就顶到输出上限。
4. **重建瘦身卡而非 unblock 旧卡**：旧卡 body 太重是崩溃根因，unblock 只会带同一份重 body 再崩。`hermes kanban archive <旧卡>`（留痕不 remove）→ 用瘦身 body 重建（`--body "$(cat body_file.txt)"`，**body 写进文件再命令替换，保证换行真实、不转义成字面 `\n`**）派 gmp。
5. **瘦身判据**：能砍的 = 每项引原文改写建议、全量逐条展开；不能砍的 = 核对维度本身（维度是验收标准，要保留），只把每项要求从「展开写」降为「判断是/否 + 一句理由」。
6. **用模板派单**：直接套 `templates/gmp_audit_body.md`（判定层 body + 字数上限 + 固定产物名 + `--body "$(cat body_file.txt)"` 传参说明），**不要再手写带 JSON 转义的单行 body**（实测：`$'...\n...'` 的换行被写成字面 `\n`，卡片 body 全塞一行）。

> 与「悬空陷阱」区分：悬空 = gmp 把修订卡派给离线 profile（monographs）；本条 = **gmp 自己**因 body 太重跑不完而崩。前者 archive 悬空卡 + oos 手动修订兜底；后者瘦身 body 重派 gmp。两种都要先 `hermes kanban show` 看 Events/Runs 判清是哪种（连续 heartbeat=正常在跑；rate_limited/crash 反复=requeue 或 body 过重；只有 created 停在 todo=派给离线 profile）。

## 派单给离线 profile 的悬空陷阱（2026-09-16 实测，最高频坑；gmp 审核 V1.0 时 2 次复现：t_49c74cf6、t_7de1106c）

gmp 审核任务 blocked 时，会自行 `create` 一张 assignee=**monographs** 的「派单留痕」卡让修订任务流转。但本 dispatcher（oos profile 这套）只在线跑 gmp/ppt 等 worker——**monographs worker 不在线，那张卡永远没人认领、悬空 todo**，链条卡死在 blocked。判别与处理：

1. **判「是否真被激活」**：`hermes kanban show <派单卡id>` 查 Events——有 `claimed`/`spawned`/`heartbeat` = 有 worker 在跑（正常）；**只有 `created` 停在 todo、无任何 claimed/spawned = 派给了离线 profile，悬空**。
2. **正确处理（oos 手动兜底，已 3 次验证）**：目标 profile 无在线 worker 时，**不要 create 给它**——oos 自己在副本上改完 V1.1，然后 4 步：① `hermes kanban archive <悬空派单卡>`（留痕不 remove）；② 在 blocked 审核卡上 `hermes kanban comment` 逐条回传「R1→已改/黄项→已改 + V1.1 路径」并注明「不派 monographs，oos 直接修订」；③ `hermes kanban unblock <审核卡>`；④ 若审核意见写明「修订后无需再审/R1 为表述更正数值链路已全量验证」，可直接 `hermes kanban complete <审核卡>` 收尾，不必再建复审卡。若仍要复审则建轻量复审卡（body 注明「仅需复核 N 处文字，无需全量重核」）。
3. **防复发**：建派单卡前先确认目标 assignee 在本 dispatcher 有在线 worker；gmp 若要派修订给非在线 profile，应在 blocked reason 里标 `needs_input` 交回人，而不是 create 离线 profile 卡。此坑在 2026-09-16 一天内 2 次复现（无效测试台账 + 偏差物料组两任务），属 gmp worker 的固定行为模式，每次 gmp 出 R/Y 意见后**默认预判它派 monographs**，主动按本段兜底、不必等卡死。

## gmp 审核意见的「可追溯性瑕疵」红项模式（2026-09-16 偏差物料组 t_7de1106c）

除分母错误外，gmp 出红项的第二类高发模式是**判定依据引用不精确（ALCOA 可追溯瑕疵，不影响统计结论）**。实例：报告把计算机化系统事件的判定依据写成「责任人[杜强]」，但源表该 sheet 列布局 col2=部门、无个人责任人列，杜强实际是「报告人」（报告人/日期列 col1）——判定本身仍成立（人∈名单），但依据标注与源表列含义不符 → 红项。修订手法：① 把该事件依据改为「报告人[杜强]（该表无责任人列，按报告人列判定）」；② 在 §0 判定口径 note 里补一句「该三源中仅此表无责任人列、按报告人/描述命中判定」，让 12 人口径的筛选逻辑与源表列含义对齐。要点：**依据文字必须逐字对应源表列名（责任人列 vs 报告人列 vs 部门列），三源列布局不同时逐源核对列语义再写依据，不能统一套「责任人[X]」措辞**。

## 偏差三源数据源的技术坑（2026-09-16 实测）

- **2026年计算机化系统事件.xlsx 的 styles.xml 损坏**：openpyxl/pandas 直接读报 `expected <class 'openpyxl.styles.fills.Fill'>`。解法：zipfile/unzip 解包 → XML 直解 `xl/worksheets/sheet1.xml` + `xl/sharedStrings.xml`（sharedStrings 里 si 可能含多段 rich text t 节点，需拼接）；sheet 单元格 `t="s"` 时值=sharedStrings 下标。**这是可复现的解析路径，不是「读不了」**。该表列布局：col1=事件编号、col2=报告人/日期（斜杠分人名与日期）、col3=部门、col4=系统、col5=主题、col6=根因、col7=关闭日期（Excel serial 需转 date）、col9=描述、col11=纠正预防、col12=CAPA编号、col13=原因类型。且该表**无个人责任人列**（只有部门），判定口径须按报告人列（见上节）。
- **25-26年偏差.xlsx 的 25年/26年 sheet 列布局不同**：25年=13列（2编号3日期4发起5责任6发起人9主题10描述11根因12类别13级别，**无 CAPA 列**→25年「无措施」是源表结构缺失、非未采取措施，报告须显式标注）；26年=31列（1编号2日期4发起人5主题6根因7类别8级别9关闭10CAPA编号24受影响产品31措施列）。「有措施」判定：26年按 CAPA编号列（有 CAPA-xx-xxx 编号=有、N/A=无），措施列(31)常年为 N/A 不作判据。
- **验证偏差.xlsx 26年 sheet 有 81 行但仅 2 条有效记录**（其余为模板空行，从 R3 起表头跨 2 行、数据从 R3 判列）：按「编号列含 V- 且非空」计有效，勿按 max_row 计数。
- 通用：**每个 xlsx 的 25年/26年 sheet 列布局都可能不同，且表头可能占 1~2 行**——解析前先用 iter_rows 打印前 2~3 行确认列名与数据起始行，再定列号，勿沿用上一年的列映射（本次 25-26偏差 25年 sheet 初版脚本误用 26年列映射导致 cause_cat 读到级别列，须回读表头校正）。

## Excel 定稿后更新 html-ppt 模板（2026-09-16 沉淀）

用户要求"更新 html-ppt 模板"时，kanban 交 ppt profile，body 必须写明：

- 新增 2–4 页 slide 的**必含内容清单**（清单表 / 趋势对比 / 根因占比 / 无效调查判定 / 重复结论 / CAPA 优先序）
- 保持原主题与图标风格；**原 slide 不删不改**
- 新增页插入位置（数据说明页之前）
- 原地更新输出目录（不新建目录）
- 完成后 summary 给**文件绝对路径 + 新增页码清单**

ppt worker 会做 section 配平 / 页码对齐 / 资源路径 preview 自检——核验时直接查 `index.html` 的 slide 数与 mtime，比等 summary 快。

## 触发词

「kanban 发给 gmp 审核」「定稿后发 ppt」「交 gmp 定稿」「做完了发起 kanban 任务」「看下 kanban 任务进度」「把报告提交给 gmp 审核」「发 kanban 任务给 gmp 审核」（单独事件 S6 终审，套 `templates/gmp_audit_body.md`；修订后复审套 `templates/gmp_s6_recheck_body.md`）

> 把整条回路**脚本化一次性建卡**（5 卡 parent 链、熔断规则、pptx/html-ppt 并行与陷阱）见 `references/scripted-auto-loop.md`。
