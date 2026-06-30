# 源码阅读 Stage 划分样例

这个样例只用于说明“源码怎么执行、怎么按代码行切 stage”，不依赖 Spark UI 或 eventlog，因此输出稳定，适合做第 1 步参考。

## 样例源码

```python
01  from pyspark.sql import functions as F
02
03  def build_feature(df):
04      # stage 1: 读取、裁剪、基础过滤
05      df_base = (
06          df.select("user_id", "item_id", "event_time", "score", "status")
07            .filter(F.col("status") == 1)
08            .withColumn("event_date", F.to_date("event_time"))
09      )
10
11      # stage 2: 关联维表并做聚合
12      df_dim = spark.table("dim_item").select("item_id", "category_id")
13      df_join = df_base.join(df_dim, on="item_id", how="left")
14      df_agg = (
15          df_join.groupBy("user_id", "category_id")
16                 .agg(
17                     F.count("*").alias("event_cnt"),
18                     F.max("score").alias("max_score")
19                 )
20      )
21
22      # stage 3: 计算衍生指标并写出
23      df_out = (
24          df_agg.withColumn("score_rate", F.col("max_score") / F.col("event_cnt"))
25                .fillna(0)
26      )
27      df_out.write.mode("overwrite").insertInto("ads_user_feature")
28
29  build_feature(spark.table("dwd_event"))
```

## 代码怎么执行

1. `build_feature(spark.table("dwd_event"))` 先把输入表变成 DataFrame。
2. `select -> filter -> withColumn` 形成第一段连续转换，属于同一个 stage 候选。
3. `join -> groupBy -> agg` 引入 shuffle，通常会切出第二个 stage 候选。
4. `withColumn -> fillna` 只是在聚合结果上做列级计算，通常仍归到第三段输出 stage。
5. `write.insertInto(...)` 是 action，触发整条链路执行。

## 推荐 stage 划分

| Stage | 代码范围 | 主要算子链 | 为什么这样切 | 状态 |
|---|---|---|---|---|
| Stage 1 | `03-09` | `select -> filter -> withColumn` | 只有窄依赖转换，直到 `join` 前都可视为同一段前处理链 | `已确认` |
| Stage 2 | `12-20` | `join -> groupBy -> agg` | `join` 和 `groupBy` 都是典型 shuffle 边界，通常会形成独立 stage | `已确认` |
| Stage 3 | `23-27` | `withColumn -> fillna -> write` | 聚合完成后进入结果整理与写出，`write` 触发 action | `已确认` |

## `源码 -> stage -> 指标` 样例

| 源码范围 | Stage | 关注指标 | 备注 |
|---|---|---|---|
| `03-09` | Stage 1 | 输入行数、过滤后行数、字段裁剪效果 | 这里重点看是否把无关列尽早裁掉 |
| `12-20` | Stage 2 | shuffle write/read、task 数、聚合耗时、是否长尾 | 这里重点看 join / groupBy 的代价 |
| `23-27` | Stage 3 | 写出耗时、输出分区数、末端整理开销 | 这里一般不是主瓶颈，但能判断写出是否异常 |

## 输出示例

```text
stage 级源码对照表

| Stage | 代码范围 | 算子链 | 风险点 | 状态 |
|---|---|---|---|---|
| Stage 1 | 03-09 | select -> filter -> withColumn | 输入字段未裁剪、过滤前置是否合理 | 已确认 |
| Stage 2 | 12-20 | join -> groupBy -> agg | shuffle 放大、是否存在倾斜 | 待确认 |
| Stage 3 | 23-27 | withColumn -> fillna -> write | 写出前是否还有冗余列 | 已确认 |

源码 -> stage -> 指标

| 源码范围 | Stage | 指标 | 说明 |
|---|---|---|---|
| 03-09 | Stage 1 | 输入行数、过滤后行数 | 关注过滤是否提前、字段是否裁剪 |
| 12-20 | Stage 2 | shuffle 读写、任务数、耗时 | 关注 shuffle 边界和聚合代价 |
| 23-27 | Stage 3 | 写出耗时、输出大小 | 关注末端整理和写入是否异常 |
```

## 这个样例的稳定点

- stage 划分依据是代码结构和 shuffle/action 边界，不依赖运行时间。
- 行号范围固定，便于文档复用。
- 输出里的 `待确认` 只对应没有 Spark UI 证据的运行指标，不会因为不同 case 波动。
- 如果真实源码比这个样例复杂，仍然按同样原则切：`窄依赖连续段 -> shuffle 边界 -> action / write`。

