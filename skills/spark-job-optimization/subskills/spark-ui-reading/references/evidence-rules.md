# Spark UI 与日志分析规则

## 目的

只收集和整理运行证据，不在这里做最终根因排序。

## 输入

- `--application_id`
- `--application_link`

## 证据采集流程

1. 如果输入是 `--application_link`，先采 ApplicationMaster 证据，再进入 Spark 运行日志。
2. 按 `jobs`、`stages`、`executors`、`environment`、`sql` 的顺序查看。
3. 从 `eventlog`、AM、driver、executor 和 YARN diagnostics 中收集可直接看到的失败或重试证据。
4. 记录每个重要 stage 的耗时、任务数、shuffle、失败、spill、preemption 和 loss reason。
5. 对于运行超过 30 分钟的 job / stage，继续进入 stage 详情页，采集 task 级证据，包括时长分布、失败 task、shuffle、spill、长尾、skew 和不均衡。
6. 如果某个 stage 的 task 太多，按 `shuffle read` 或 `shuffle write` 排序，只保留第一页 task 详情。
7. 如果某个 stage 读取超过 10 亿行或 2T 输入，先标记为规模检查候选，继续核对分区裁剪、字段裁剪、重复扫描，以及是否需要引入中间表。
8. 如果输入不大但文件数很多，或者平均单文件大小小于 16M，先标记为小文件候选。
9. 如果运行证据能直接对应到某条 SQL，要把 SQL 原文和执行计划一起捕获，并标明它对应的 stage。
10. 尽量保留原始表格结构，优先输出 Markdown 表格，不要把表格压成纯文本。

## 必须采集的证据

- 应用概览：application id、name、user、queue、状态、开始结束时间、diagnostics
- 环境：Spark 版本、Spark 配置、执行参数、dynamic allocation、shuffle partitions、broadcast threshold、AQE、speculation
- Jobs：数量、长耗时 job、失败或重试 job、重复提交痕迹
- Stages：耗时、task 数、输入输出、shuffle read/write、失败原因、retry/skipped/spill
- Executors：active/dead/total、负载平衡情况、GC/spill/shuffle/task time、loss reason
- SQL：query plan、join/exchange/window/broadcast/stage 映射
- EventLog：原始 stage/task/job 证据
- AM / driver / executor / YARN diagnostics：`OOM`、`Killed by YARN`、`preempted`、`node lost`、`fetch failed`、`file not found`、`disk error`、`exit code`

## 标签规则

标签只能基于 Spark UI、运行日志或 event log 中直接可见的证据。

### 优先级较高的日志判断标签

下面这些标签适合先做第一轮自动判定，再决定要不要继续深挖：

- `data_skew_label`：shuffle read / write 存在明显倾斜，优先检查热点 key、极端大分区、长尾 task
- `compute_waste_label`：运行时间过长、内存分配明显偏大、GC 偏高或 spill 明显，优先检查资源浪费和执行参数是否失配
- `runtime_gt_1h_label`：运行时长超过 60 分钟，优先进入长链路、重复扫描、超大 stage 的判断
- `submit_wait_gt_10min_label`：排队或提交等待超过 10 分钟，优先检查队列拥塞和资源可用性
- `memory_waste_label`：分配内存显著大于输入规模，优先检查是否存在过配或错误的资源申请
- `memory_alloc_insufficient_label`：分配内存小于输入规模，优先检查是否存在明显低配
- `data_skew_read_label` / `data_skew_write_label`：用于定位 shuffle 读 / 写倾斜的具体方向
- `full_gc_label`：major GC 占比偏高，优先检查对象膨胀、缓存压力和过度分配
- `spill_disk_label`：存在明显 spill disk，优先检查 executor memory、partition 大小和 join / sort / agg 路径

### 失败与重试

- `task_failed_label`：`final_state = failed`
- `spark_stage_failed_label`：Spark 任务失败或 `attempt > 0`
- `spark_stage_failed_gt3_label`：Spark 任务失败或 `attempt > 3`
- `spark_task_retry_label`：Airflow 或 task 存在重试
- `spark_task_retry_gt3_label`：重试次数 > 3

### 时长与排队

- `runtime_gt_1h_label`：运行时长 > 60 分钟
- `submit_wait_gt_10min_label`：排队 / submit wait > 10 分钟
- `mu_top100_label`、`mu_top50_label`、`runtime_top100_label`、`runtime_top50_label`：按日或近 7 天的资源消耗 / 运行时长排名标签

### 内存分配

- `memory_alloc_insufficient_label`：平均分配内存小于输入规模
- `memory_waste_label`：平均分配内存显著大于输入规模

### 倾斜与长尾

- `data_skew_read_label`：最大 shuffle read 与平均 shuffle read 差距显著
- `data_skew_write_label`：最大 shuffle write 与平均 shuffle write 差距显著
- 如果能明显看到时长长尾、P95 或 P99，可作为 skew / long-tail 观察项

### GC 与 Spill

- `full_gc_label`：major GC 占 task time 的比例过高
- `spill_disk_label`：存在明显 spill disk

### 任务健康

- `task_health_label`：当 failed / retry / rank / memory / skew / GC / spill 等异常标签都不触发时使用

## 分析约束

- 不在这里做最终根因排序。
- 不写没有直接证据支撑的因果结论。
- 如果某个值没有直接暴露，就写 `待确认` 或 `未直接给出`。
- `ExecutorLostFailure`、`preempted`、`killed`、`fetch failed` 先按症状记录，除非日志直接证明它就是根因。
- 如果总数据量不大但某个 stage 很慢，要继续往 skew、长尾、大分区、热点 key 方向查。
- 如果窗口排序、groupBy、join 这类 shuffle 慢点出现，先判断是总量大还是倾斜大。
- 如果只看到平均值，不要把平均值当成现场，优先记录 P95、P99 或最慢 task。
- 如果某个 stage 读取超过 10 亿行或 2T 输入，先标成规模异常候选。
- 如果读取数据量不大但文件数很多，或者平均单文件大小小于 16M，先标成小文件候选。
- 如果超大 stage 数量超过 4 个，优先提示是否需要把部分 stage 落表或拆成中间表。

## 输出

- 集群和环境信息
- Jobs / stages / executors / SQL 证据
- AM / driver / executor / YARN diagnostics 证据
- 每个关键 stage 的运行备注
- 需要带入根因分析阶段继续处理的证据缺口
- 每个关键 stage 的 task 数、耗时、shuffle、失败、spill、preemption、loss reason
- 和源码 stage 对齐的运行证据表

## 约束

- 不在这里排序根因。
- 不写没有证据支撑的因果结论。
- 保留原始证据，不要改写成摘要。
- 如果输入是 `--application_link`，必须先采 ApplicationMaster。
- 运行日志分析必须覆盖 `jobs`、`stages`、`executors`、`environment`、`sql`，并尽量补充 `eventlog`、AM、driver、executor 和 YARN diagnostics。
