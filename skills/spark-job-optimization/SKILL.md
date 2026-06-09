---
name: spark-job-optimization
description: 用于通过源码和 Spark UI 日志分析 PySpark 任务，定位瓶颈并按确认后的顺序执行优化。
---

# Spark Job Optimization

## 概述

当 PySpark 任务变慢时，按固定顺序处理：先读源码，再读执行证据，确认主要原因，确认可改点，最后再改代码。

## 输入约定

- 每个分析案例放在 `input/<case_name>/`。
- 源码 zip 放在 `input/<case_name>/source.zip`，或者解压到 `input/<case_name>/source/`。
- Spark UI 导出放在 `input/<case_name>/spark_ui/`。
- 集群和上游表上下文放在 `input/<case_name>/context/`（如有）。
- event log、SQL plan、截图、运行备注放在同一个 case 目录里。
- 诊断报告和变更产物输出到 `output/<case_name>/`。
- 如果 case 名不明显，就用主任务名或主表名。

## 流程

0. 采集集群资源和上游表上下文。
   使用 `scripts/collect_case_context.py` 和 `references/context-collection.md`。
   如果 Spark UI 只能通过登录态浏览器访问，先用 `scripts/collect_spark_ui_browser.py` 采集页面内容（优先保留表格块），再跑 `collect_case_context.py`。
1. 调用源码分析子 skill，形成最终报告第 1 章素材。
   见 `subskills/source-reading/SKILL.md` 和 `references/step-1-source-reading.md`。
2. 调用 Spark UI / 日志分析子 skill，形成最终报告第 2 章素材。
   见 `subskills/spark-ui-reading/SKILL.md`、`references/step-2-spark-ui-reading.md` 和 `references/input-contract.md`。
   这一阶段同时按 `references/spark_ui_label_definition.sql` 的同级口径整理失败、重试、长时、排队、skew、full GC、spill、内存不足 / 浪费和任务健康标签。
3. 结合代码路径和运行证据定位慢点，形成最终报告第 3 章素材。
   见 `references/step-3-root-cause-ranking.md` 和 `references/diagnostic-heuristics.md`。
4. 按影响度排序前 5 个有证据支撑的瓶颈，给出直接支持的优化建议、预期收益估算和对应的代码调整片段或代码草图，并等待用户确认哪些可以改，形成最终报告第 4 章。
   见 `references/step-4-optimization-proposal.md`、`references/final-report-spec.md` 和 `references/final-report-template.md`。
   第 4 步报告文字版和图片都要尽量贴近 `references/step-4-report-template.md` 与 `references/assets/step4-report-sample.svg`，并且图片左侧必须按 Spark stage 用虚线框拆分完整源码，右侧用放大镜卡片展示瓶颈原因和优化方向。
5. 根据用户人工 review 后确认的内容，执行已确认的代码变更，并产出第 5 章素材。
   见 `references/step-5-code-change.md`。
   这一阶段必须保留优化前 / 优化后两个代码快照，不使用 git 历史充当版本记录；同时要产出源码 -> 代码变更 / 代码调整映射图。
6. 重新采集优化后 Spark UI / 日志，并与优化前快照对比，形成最终报告第 6 章素材。
   见 `references/step-6-final-verification.md`。
   优化后快照默认放在 `spark_ui/optimized_browser/`，对比规则见 `references/input-contract.md` 和 `references/step-2-spark-ui-reading.md`。
   这一阶段要产出源码 -> 执行情况映射图、优化前后对比表和最终效果结论；如果这次优化验证出了新的通用规律，要同步沉淀到 `references/diagnostic-heuristics.md` 和 case 的 `notes.md`。

## 扩展点

默认保持 skill 简单。如果后面某一步变大了，再把那一步拆到 `references/` 里，这个文件只保留流程入口。

## 输出原则

- 只有证据可直接观测时才写“已确认”；如果日志没有暴露，就写 `待确认`，不要猜。
- 正文是主载体，图片是细节展开；第 4/5/6 章都必须先在正文里写清楚映射关系和结论，再用图片补局部细节。
- Executor 丢失、抢占、重试、Killed attempt 先当作症状，除非执行日志直接证明它是根因。
- 优先给能先去掉主瓶颈的最小改动，但没有直接证据前不要把改动写成已验证。
- 排序优化思路时，顺序固定为：先调 Spark 参数，再改实现方式，最后才考虑改业务逻辑。
- 源码分析必须按 Spark stage 拆分，形成 `源码 -> stage -> 指标` 对照；看到任何算子都要能回指到对应 stage 的运行信息。
- 最终报告固定按六章输出：源码分析、运行日志分析（优化前）、根因分析、优化方案、优化代码变更（人工确认后）、优化效果验证（优化后）；每一章只写自己那一步必须产出的内容。
- 第 4 章正文必须先给出 `优化项 -> 代码片段 / 代码草图 -> 目标 stage -> 运行证据 -> 预期收益` 映射表，再用图片展示热点细节。
- 第 5 章正文必须先给出 `源码 -> 代码变更` 映射表，再用图片展示局部 diff。
- 第 6 章正文必须先给出 `源码 -> 执行情况` 映射表，再用图片展示前后运行差异。
- 第 5 章必须输出源码 -> 代码变更映射图，供人工确认后的变更落地审阅。
- 第 6 章必须输出源码 -> 执行情况映射图，供优化前后验证与效果复盘使用。
- 优化完成后，不只是总结效果，还要把本次验证出来的新经验回写到诊断 heuristics 或 case 备注里，形成可复用规则。
- 优化完成后，代码也必须保留优化前 / 优化后两个版本快照，不依赖 git 历史作为版本管理。
- 分析源码时，额外关注任何会把一条记录展开成多条记录的操作，尤其是 `explode`、`explode_outer`、`flatMap`、`posexplode`、数组展开和展开后再聚合 / join 的链路。
- 分析源码时，额外关注中间列的下游活跃性；像 `select`、`withColumn`、`alias`、`rank`、`row_number` 这类中间结果，如果后面没进入 `filter`、`join`、`groupBy`、`agg` 或 `write`，要当作冗余列处理；如果短期内不方便删干净，也可以考虑在链路一开始把对应字段置空或置默认值，但前提是不会影响中间排序、过滤或写入口径。
- 分析源码时，额外关注上游出现、下游未使用、最终也未写入结果表的字段；如果确认不影响结果口径，优先按冗余字段候选处理并考虑删除；必要时也可以用前置置空 / 置默认值作为过渡方案。
- 除了最终写入或者明确的验证动作，尽量不要在中间链路频繁使用 `show()`、`collect()`、`take()`、`toPandas()` 这类 action。
- 如果某个优化项暂时没有明确的最终代码，也必须给出代码草图或局部 diff，不能只留空或只写 `待确认`。
- 如果某个优化项没有精确收益数字，也必须给出预估逻辑和高 / 中 / 低 级别判断，不能只写空的 `待确认`。
- 没有直接证据支撑的收口动作、末端写出动作或常规输出动作，不要单独写成风险；如果确实要写风险，必须说明具体影响是什么。
- 第 4 章图片里，同一热点的卡片之间必须保留足够间距，后一张卡的标题不能覆盖前一张卡的内容。
- 优化完成后，要把本次验证出来的新经验沉淀回 `diagnostic-heuristics.md`，并在最终报告第 6 章里总结优化效果和可复用经验。
