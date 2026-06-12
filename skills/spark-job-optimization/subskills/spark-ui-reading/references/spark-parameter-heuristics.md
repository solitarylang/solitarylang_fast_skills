# Spark 参数解读与调参经验

## 目的

把 Spark 运行参数和实际运行表现对齐，沉淀“参数配置是否和日志现象相符”的经验口径。

这份规则只做两件事：

- 解释关键 Spark 参数在当前任务里的实际含义
- 当参数配置和运行日志、Spark UI 现象冲突时，记录为调参候选，而不是直接当作根因

## 先看哪些参数

优先看下面这些参数，因为它们最容易和运行日志形成直接对应关系：

- `spark.executor.memory`
- `spark.executor.memoryOverhead`
- `spark.executor.cores`
- `spark.executor.instances`
- `spark.dynamicAllocation.enabled`
- `spark.dynamicAllocation.minExecutors`
- `spark.dynamicAllocation.initialExecutors`
- `spark.dynamicAllocation.maxExecutors`
- `spark.sql.shuffle.partitions`
- `spark.sql.adaptive.enabled`
- `spark.sql.adaptive.skewJoin.enabled`
- `spark.sql.autoBroadcastJoinThreshold`
- `spark.sql.broadcastTimeout`
- `spark.speculation`
- `spark.memory.fraction`
- `spark.memory.storageFraction`

## 常见解释口径

### `spark.executor.memory`

- 这是单个 executor 的 JVM 堆内存。
- 如果日志里频繁出现 `spill`、`full GC`、`OOM`、task 内存抖动，先检查它是否偏小。
- 如果 executor 长时间大面积空闲，或者 GC、spill 都不明显，但内存配置很大，要考虑是否过配。

### `spark.executor.memoryOverhead`

- 这是堆外和额外进程开销的缓冲区。
- 如果 driver / executor 日志里出现 native memory、off-heap、PySpark、shuffle 相关异常，但堆内存看起来还够，优先检查它。

### `spark.executor.cores`

- 这是单个 executor 并发跑 task 的数量上限。
- 如果单 executor CPU 被打满、task 排队明显、总 executor 数不多但 stage 仍然很慢，要检查是不是 cores 偏少。
- 如果单个 executor 上 task 争用严重、GC 和 CPU 上下文切换偏高，也要考虑 cores 偏多导致并发过度。

### `spark.executor.instances` 与 dynamic allocation

- `spark.executor.instances` 决定静态 executor 数。
- 如果 `spark.dynamicAllocation.enabled = true`，重点看 `min / initial / maxExecutors` 是否限制了扩缩容。
- 如果 stage 很大、active tasks 很多，但 executor 数长期上不去，优先怀疑 `maxExecutors` 偏小或资源池受限。
- 如果 executor 数明显偏多，但大部分时间活跃 task 不多，要考虑是否过配。

### `spark.sql.shuffle.partitions`

- 这是 shuffle 后分区数的核心控制项。
- 如果 task 数过少、单 task 过重、长尾明显，分区数可能偏少。
- 如果 task 数太多、每个 task 很短、调度开销明显、stage 被切得过碎，分区数可能偏多。

### `spark.sql.adaptive.enabled` 和 `spark.sql.adaptive.skewJoin.enabled`

- AQE 打开后，很多固定分区问题会被动态修正。
- 如果 AQE 已经打开，但依然看到明显 skew、长尾、极端大分区，要优先记录为“AQE 未完全覆盖的倾斜候选”，不要直接假设 AQE 会自动修掉。

### `spark.sql.autoBroadcastJoinThreshold`

- 如果小表 join 仍然走大 shuffle，先看广播阈值是否太小、表统计是否缺失、或者广播条件是否没有被满足。
- 如果广播表过大，导致 driver 压力、executor 内存压力或广播超时，要考虑阈值过大。

### `spark.speculation`

- 如果日志里有大量 `killed: another attempt succeeded`、重复 attempt、长尾 task 重跑，要把 speculation 是否开启、是否放大重复计算纳入判断。

## 参数与日志冲突时怎么判断

当配置和运行现象冲突时，优先相信运行证据，再判断参数是否“失配”：

1. 先看 Spark UI / event log / driver / executor 日志里直接暴露的现象。
2. 再看参数是否能解释这个现象。
3. 如果参数解释得通，就记录为“参数调优候选”。
4. 如果参数解释不通，就继续往数据量、倾斜、重复扫描、业务逻辑方向查。

## 常见冲突模式

- `executor.memory` 很大，但仍然频繁 `spill` / `full GC`：优先怀疑数据倾斜、过大分区、对象膨胀或 shuffle 压力，而不是单纯继续加内存。
- `executor.memory` 很大，但大部分 executor 长期空闲：优先怀疑过配。
- `executor.cores` 很少，但 stage 并行度高、task 排队明显：优先怀疑并发不足。
- `executor.instances` 很少或 `maxExecutors` 很小，但 active tasks 很多：优先怀疑并行度受限。
- `shuffle.partitions` 很大，但每个 task 很轻、调度时间占比高：优先怀疑分区过碎。
- `shuffle.partitions` 很小，但 task 长尾和单点重计算明显：优先怀疑分区过粗或 skew。
- `broadcast` 没有触发，但小表 join 很明显：优先怀疑广播阈值、统计信息或 join 写法。
- `AQE` 已开启，但 skew 仍然明显：优先怀疑 AQE 未覆盖、数据倾斜太重或上游已经把分布放大了。

## 输出要求

在 `spark-ui-reading` 结果里，参数解读要明确记录：

- 这组参数是什么
- 它和当前运行证据是否一致
- 如果不一致，冲突点是什么
- 这是不是一个可验证的调参候选
- 需要修改哪类参数，而不是只写“参数有问题”

## 不做什么

- 不直接把参数失配等同于根因
- 不在没有日志证据时单靠经验做参数修改结论
- 不把“建议调参”写成“已经验证有效”
