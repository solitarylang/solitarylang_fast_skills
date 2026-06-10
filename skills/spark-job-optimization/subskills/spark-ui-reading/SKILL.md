---
name: pyspark-spark-ui-reading
description: 用于独立收集 Spark UI、YARN、driver、executor 和 eventlog 证据，输出第 2 章素材。适用于只需要运行证据采集、环境信息和失败链路分析的场景。
---

# Spark UI 与日志阅读

## 参数约定

优先通过显式参数传入案例上下文。只保留两个入参：

- `--application_id`：YARN / Spark 应用 id，优先使用。
- `--application_link`：应用链接，优先支持 YARN application 链接，也可用于 Spark UI 链接。

参数优先级如下：

1. 先用 `--application_id` 锁定运行实例。
2. 如果没有应用 id，则用 `--application_link` 作为入口。

只允许传入一个主参数；如果同时传入多个来源，以 `--application_id` 为准。

## 执行流程

1. 根据参数读取案例输入，来源包括 `--application_id` 或 `--application_link`。
2. 按 `jobs`、`stages`、`executors`、`environment`、`sql` 的顺序查看证据。
3. 对于重要或长时间运行的 job / stage，继续进入 stage 详情页，采集 task 级证据。
4. 采集完证据后，结合 `references/execution-abnormality-rules.md` 判断当前执行是否存在不合理现象。
5. 只记录直接证据，并保持 stage 到源码的对齐关系明确。
6. 停止在根因排序之前，把根因分析留给父级优化 skill。

## 参考

`references/evidence-rules.md`：输入约定、证据清单、标签规则、输出要求和分析约束。  
`references/input-parameters.md`：参数说明、优先级和输入组合方式。  
`references/execution-abnormality-rules.md`：执行不合理的定义，以及后续可扩展的场景槽位。
