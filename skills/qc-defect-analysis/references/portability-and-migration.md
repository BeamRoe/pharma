# 跨公司迁移：references 可移植性分类与一次执行适配法

> 触发场景：用户要把本 skill 搬到另一家公司 / 环境，问"哪些能用、哪些要重建、一次执行够不够"。
> 核心结论：**一次 skill 执行即可适配新环境，不需要手改全部 references**——前提是把"绑死本公司"的内容从正文剥离、归为可重建的产物，方法论留可迁移判据。

## 三类判定（每个 reference 按"是否硬编码本公司事实"归入）

| 档 | 判定标准 | 迁移处理 | 本 skill 实例 |
|---|---|---|---|
| 通用方法 | 只含方法/规则/模板，举例不含或几乎不含公司专属 | 直接带走，不改 | data-model / html-output-patterns / capa-effectiveness / capa-quality-assessment / oos-deviation-severity-inference / report-sheets / cross-ledger-person-trend / workload-correlation-analysis |
| 方法通用·举例带公司数据 | 方法可复用，但举例里的 sheet 名/列号/人名/房间是本公司的 | 保留方法，把举例里的公司专属值换成新公司对应值 | data-parsing-pitfalls / defect-classification-taxonomy / invalid-test-analysis-patterns / oos-event-id-and-source-reconciliation / field-cleaning-and-scoping |
| 绑死本公司 | 整文件硬编码本公司台账结构（列号、房间清单、sheet 名、房间→班组映射） | 删掉，新环境重新探测生成 | column-mapping-guide |

**判定手法（可复用）**：对每个 reference 跑信号扫描，看"硬信号"占比——
- 固定房间清单（`一般理化室|TOC室|气相色谱室|天平室|…`）
- 年份 sheet 名（`24/25/26年自检问题汇总`、`25年-1/2/3/4`）
- 列号硬编码（`col0`/`col3`/`第N列`）
- 公司 SOP/SMP 编码（`SMP-EE-xxxx`/`SOP-07-xxxx`/`QC0xxx`）
- 用户环境路径（`/home/…`、`~/桌面`、`~/文档`）
- 具体人名（脱敏名也算，如 `蒋某某`/`扎西bb`）

**判档规则**：硬信号占全文件行数 < 10% 且只是举例 → 归"方法通用"档；举例占多数但方法通用 → 归"局部重写"档；整文件=某公司台账结构手册 → 归"绑死"档。**别只看信号个数，要看"删掉这些信号后方法还成不成立"。**

## 一次执行完成适配的流程（给新公司用户）

1. 删绑死档文件（如 `column-mapping-guide.md`）
2. 正文"已知陷阱"里**公司专属**的条（某年无某列、某 sheet 某列全空、具体人名/房间映射）移入归档 `references/known-pitfalls-archive.md`，正文只留可迁移判据
3. 对"局部重写"档，把举例里的 sheet 名/列号/人名替换为新公司值（方法逻辑不动）
4. 跑一次任意新台账 → 触发 **Step 0.5 新环境列结构探测**：逐 sheet 打印前 5 行+表头 → 按表头自动推列索引 → 写新文件 `references/company-<代号>-column-map.md`
5. 按新台账重新校准 `defect-classification-taxonomy.md` 的占比参考（占比是公司样本统计值，换公司要重测）

> 适配动作是"探测并生成新映射文件"，不是"手改 15 个 reference"。映射文件是**产物**不是 skill 本体，新文件按新公司代号命名，旧的留着无害。

## 与正文的衔接

- Step 0.5 探测是本 skill Phase 1 的关 3 在新环境下的落地动作；没有它，column-mapping-guide 删掉后新环境无列映射依据。
- 迁移/打包清单本身（哪几个文件删/改/带走 + 正文同步改哪几行）是**一次性产物**，交给用户打包，不进 references（避免与"一次一产物"冲突）；本文件只固化**判定方法与流程**。

## 防误用（与 memory/AGENTS 的边界）

迁移判据属于"类级方法"，归本 skill；memory 只存"当前这份数据在哪、口径怎么定"。二者别混：把"本公司 24 年无班组列"写进 memory 是错的——那是公司专属事实，应在 skill 的归档陷阱或公司映射文件里。
