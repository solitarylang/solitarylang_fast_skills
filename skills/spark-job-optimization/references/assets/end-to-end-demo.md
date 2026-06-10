# Spark 作业优化端到端样例

这是一个**完整演示样例**，用于展示 `spark-job-optimization` skill 从输入、分析、提案、变更到验证的全流程输出格式。

- 这是结构示例，不是实际线上 case 结论。
- 数值、SQL、代码片段均为示意，用来说明每一步应该输出什么。
- 如果后续实际 case 证据更少，可以缩减内容，但不要改变信息层级。

## 0. 样例输入

```text
<business_repo_root>/tmp/spark-job-optimization/sample_case/
  input/
    source.zip
    source/
    spark_ui/
      browser/
    context/
      context_report.md
    eventlog/
    notes.md
  tmp/
  output/
    report/
    figures/
  log/
```

## 1. 第 1 步：源码分析输出

### 1.1 目标

把源码按 Spark stage 拆开，形成 `源码 -> stage -> 指标` 对照。

### 1.2 输出样例

| Stage | 代码范围 | 算子链 | 风险点 | 状态 |
|---|---|---|---|---|
| Stage 1 | `job.py:18-64` | `scan -> filter -> withColumn -> row_number` | 360 天全扫，窗口排序重 | `已确认` |
| Stage 2 | `job.py:65-118` | `explode -> groupBy -> agg` | 增行后再聚合，shuffle 放大 | `已确认` |
| Stage 3 | `job.py:119-154` | `join -> write` | 宽 join，末端收口 | `待确认` |

### 1.3 关键信号

- `row_number` 结果下游未消费，属于冗余中间列候选
- `explode` 后紧跟 `groupBy`，属于典型增行放大链路
- 源表为长历史分区表，存在重复 scan 候选

### 1.4 输出结论

- 已确认：Stage 1 和 Stage 2 是主慢路径候选
- 待确认：Stage 3 的 join 是否为主瓶颈，需要 Spark UI 补证据

## 2. 第 2 步：Spark UI / 日志分析输出

### 2.1 集群与环境

| Key | Value |
|---|---|
| `spark.executor.instances` | `24` |
| `spark.executor.cores` | `4` |
| `spark.executor.memory` | `16g` |
| `spark.sql.shuffle.partitions` | `800` |
| `spark.sql.adaptive.enabled` | `true` |
| `spark.sql.autoBroadcastJoinThreshold` | `10MB` |

### 2.2 Jobs / Stages / Executors

| Type | Id | 证据 |
|---|---|---|
| Job | `16` | `4.2 h`，主慢 job |
| Stage | `1` | `6.4 TiB` 输入，`129.4 GiB` shuffle write，`61,870` tasks |
| Stage | `2` | `shuffle read` 明显放大，task 长尾明显 |
| Executors | `dead` | 有少量 `killed: another attempt succeeded`，更像慢任务外显 |

### 2.3 SQL / EventLog / Driver

- SQL 原文已定位到 `window + explode + groupBy` 组合链
- EventLog 里能看到 Stage 1 先重 scan，再进入窗口排序
- Driver 日志未暴露明确 OOM，不能把 executor 丢失直接当根因

### 2.4 运行链路结论

- 主慢点不是单一失败，而是大 scan + 增行 + shuffle 的组合放大
- 任务长尾和少量失败更像慢任务伴随症状

## 3. 第 3 步：根因排序输出

### 3.1 排名样例

| 排名 | 根因 | Stage | 证据强度 | 状态 |
|---|---|---|---|---|
| 1 | 360 天全量扫描 | Stage 1 | 高 | `已确认` |
| 2 | `row_number` 冗余计算 | Stage 1 | 中 | `待确认` |
| 3 | `explode -> groupBy` 数据膨胀 | Stage 2 | 高 | `已确认` |
| 4 | 重复 shuffle / 宽 join 收口 | Stage 3 | 中 | `待确认` |
| 5 | 参数未收紧导致 shuffle 过碎 | Stage 1/2 | 中 | `待确认` |

### 3.2 排序原则

1. 先排能覆盖最大耗时链路的问题
2. 先排有直接证据的问题
3. 先排能用最小改动解决的问题

## 4. 第 4 步：优化方案输出

### 4.1 参数调优候选

| 优化项 | 代码片段 / 代码草图 | 目标 Stage | 运行证据 | 预期收益 |
|---|---|---|---|---|
| 调整 shuffle 分区 | `spark.sql.shuffle.partitions = 400` | Stage 2 | shuffle 任务过碎 | 中 |
| 打开 / 保持 AQE | `spark.sql.adaptive.enabled = true` | Stage 1-3 | shuffle 过大且长尾明显 | 中 |

### 4.2 实现方式优化

| 优化项 | 代码片段 / 代码草图 | 目标 Stage | 运行证据 | 预期收益 |
|---|---|---|---|---|
| 裁掉冗余列 | 删除未消费的 `row_number` 计算 | Stage 1 | 下游未使用 | 中 |
| 拆掉 `explode -> groupBy` 膨胀链路 | 前置过滤 / 预聚合 | Stage 2 | 行数放大 | 高 |
| 引入中间表 | 把长历史 scan 结果落地复用 | Stage 1 | 360 天全扫 | 高 |

### 4.3 业务逻辑优化

| 优化项 | 代码片段 / 代码草图 | 目标 Stage | 运行证据 | 预期收益 |
|---|---|---|---|---|
| 收窄历史窗口 | `where(pt_date >= '<start>')` | Stage 1 | 长历史分区全扫 | 高 |

### 4.4 待确认项

- `row_number` 是否对最终口径有真实影响
- Stage 3 join 是否能通过广播或字段裁剪继续收敛

## 5. 第 5 步：代码变更输出

### 5.1 人工确认结果

- 批准删除冗余 `row_number` 计算
- 批准把长历史 scan 前置为中间表
- 暂不改业务逻辑，只做实现方式优化

### 5.2 变更示意

| 源码范围 | 变更前 | 变更后 | 说明 |
|---|---|---|---|
| `job.py:18-64` | 计算 `row_number` 后继续透传 | 删除该列或改为前置空值 | 冗余中间列 |
| `job.py:18-64` | 直接全量扫描 | 先落中间表再复用 | 降低重复 scan |
| `job.py:65-118` | `explode -> groupBy` | 先过滤再展开 | 减少膨胀 |

### 5.3 快照要求

- 优化前：`input/source_before/`
- 优化后：`input/source_after/`

## 6. 第 6 步：优化效果验证输出

### 6.1 优化前后对比

| 指标 | 优化前 | 优化后 | 变化 |
|---|---:|---:|---|
| 总运行时长 | `4.2 h` | `58 min` | 明显下降 |
| 主慢 Stage 输入 | `6.4 TiB` | `113 GiB` | 大幅下降 |
| shuffle write | `129.4 GiB` | `21.8 GiB` | 显著下降 |
| 失败 task | `300` | `12` | 明显下降 |

### 6.2 结论

- 主耗时链路已经从“大 scan + 增行 + shuffle”收缩为可控的中间表复用链路
- 少量失败和长尾被削弱，但仍建议继续观察 Stage 3 的 join 稳定性

### 6.3 经验沉淀

- 长历史全量 scan 优先考虑中间表或增量化
- `explode -> groupBy` 需要优先检查是否存在前置过滤空间
- 下游未消费的中间列，优先删除计算逻辑，不要只在末端 drop

## 7. 这个 demo 告诉你什么

这套 skill 的完整闭环应该输出：

1. `第 1 步` 先把源码和 stage 对齐
2. `第 2 步` 用运行证据验证慢点
3. `第 3 步` 把根因排序
4. `第 4 步` 给出可执行优化方案
5. `第 5 步` 记录人工确认后的最小变更
6. `第 6 步` 用优化前后对比验证收益，并回写新经验
