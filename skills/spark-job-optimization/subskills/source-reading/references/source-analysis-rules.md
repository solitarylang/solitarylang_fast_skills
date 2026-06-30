# PySpark 源码分析规则

## 目的

只做源码阅读，不进入运行日志归因。目标是把代码路径按 Spark stage 拆开，形成 `源码 -> stage -> 指标` 对照。

## 输入

- `<business_repo_root>/tmp/spark-job-optimization/<case_name>/input/source.zip` 或 `.../input/source/`
- `<business_repo_root>/tmp/spark-job-optimization/<case_name>/input/context/context_report.md`（如有）
- `<business_repo_root>/tmp/spark-job-optimization/<case_name>/input/spark_ui/browser/` 或 `input/eventlog/` 中可用于 stage 对齐的证据

## 工作方式

1. 先定位入口文件、主类 / 主函数和调用参数。
2. 再识别主链路和分支链路，找出所有 DataFrame / RDD / action 的落点。
3. 按 Spark stage 切分代码路径。
4. 对每个 stage 记录对应的算子链、输入输出、增行点、shuffle 点、冗余列和重复聚合。
5. 如果某段源码还不能明确映射到 stage，就标成 `待确认`。

## Stage 划分硬规则

| 规则 | 要求 | 说明 |
|---|---|---|
| 一一对应 | 每个算子只能归属一个 stage | 不能把同一个算子同时写进两个 stage，也不能漏掉任何算子 |
| 连续闭合 | stage 代码范围必须是连续区间 | 不允许跳段拼接，不允许把中间代码空出来 |
| 边界准确 | stage 边界优先落在 shuffle / action / write / 明确的算子断点上 | 先找边界，再填 stage 内算子 |
| 表内一致 | `代码范围`、`算子链`、`风险点` 三者必须互相对应 | 不允许 stage 表和源码行号对不上 |
| 不重叠不遗漏 | 相邻 stage 之间不得重叠，也不得留空白算子 | 若出现无法判断的边界，标 `待确认`，不要硬拆 |

## 需要重点识别的源码信号

| 分类 | 需要检查的信号 | 处理建议 |
|---|---|---|
| 入口与链路 | 入口 / 出口点、主数据流和分支数据流、每个 action 的触发位置 | 先确认入口，再把主链路和分支链路画出来 |
| 重复计算 | 重复读取、重复 join、重复聚合、重复扫描；跨日期重复解析同一批历史数据 | 优先标记为重复执行候选，检查是否可增量化、按天中间表化或复用解析结果 |
| 列级变换 | `withColumn` / `select` / `alias` / `drop` 这类列级变换；排序字段、`rank` / `row_number` / `dense_rank` 这类下游未消费的列 | 优先识别冗余中间列，能删则删，不能删再考虑末端 drop |
| 增行点 | `explode` / `explode_outer` / `flatMap` / `posexplode` 这类增行点 | 作为数据膨胀热点单独标记 |
| Action 点 | `count()` / `show()` / `collect()` / `take()` / `toPandas()` | 判断是否会触发全量执行，避免把 action 当普通转换看待 |
| 大扫描 / 大分区 | 单个 stage 读取超过 10 亿行或 2T；窗口排序、groupBy、join 这类 shuffle 慢点；大分区操作 | 先判断总量问题还是倾斜问题，再决定是否落表、裁剪或重构 |
| 中间态复用 | scan 很大但到第一个 shuffle 前 `read / input` 明显收缩；scan 后紧跟大量解析 / 映射 / 标准化 / 分支归一化 | 优先考虑前置中间表、增量中间表或中间快照 |
| 小文件问题 | 读取量不大但文件数很多，或者平均单文件大小小于 16M | 优先标成小文件风险候选，检查上游合并与写入粒度 |
| 冗余字段 | 上游出现、下游未使用、最终也未写入结果表的字段 | 优先评估删除，无法删除时再考虑保留方案 |

## 输出

| 输出项 | 说明 |
|---|---|
| `源码 -> stage -> 指标` 对照表 | 作为最终结构化结果的核心表 |
| 按 stage 拆分的源码分析结果 | 包含代码范围、算子链路、数据流和 action 点 |
| 每个 stage 的风险点与观察项 | 包含增行、shuffle、skew、冗余列和冗余分区 |
| 每个 stage 的 `已确认` / `待确认` 标记 | 便于后续和日志证据对齐 |
| 每个 stage 的小文件风险候选 | 若文件数很多或平均单文件小于 16M，要明确写出 |
| 每个 stage 对应的数据文件数 / 平均文件大小 / 可推导证据 | 作为小文件判断的依据 |
| 需要补齐的证据缺口 | 留给后续运行日志分析阶段 |

## 必须产出

| 必须产出 | 说明 |
|---|---|
| Stage 级源码对照表 | 代码范围与 stage 的对应关系 |
| 关键算子链路说明 | `scan / filter / join / groupBy / write` 等关键链路 |
| 增行 / shuffle / skew / 冗余列 / 冗余分区风险点 | 需要单独标注的分析信号 |
| 冗余字段可删候选 | 下游未消费且最终未写入结果表时要明确标记 |
| Action 行为判断 | `count()` / `show()` / `collect()` / `take()` / `toPandas()` 是否会触发全量执行 |
| 需要补齐的运行证据 | 留给后续阶段补采 |

## 约束

- 不直接做根因排序。
- 不直接输出优化建议。
- 不把 Spark UI 的结论提前写成源码结论。
- 每个算子都要尽量能回指到一个 stage。
- 源码分析必须按 Spark stage 拆分，而不是只按文件或函数拆分。
- stage 划分必须和算子边界一一对应，不能出现同一算子跨多个 stage、一个 stage 漏算子或阶段重叠。
- 看到一个算子时，必须能回指到它所在的 stage、该 stage 的运行时长、任务数、输入输出和 shuffle 指标；如果不能回指，就先标 `待确认`。
