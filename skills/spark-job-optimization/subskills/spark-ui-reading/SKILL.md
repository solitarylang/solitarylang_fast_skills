---
name: pyspark-spark-ui-reading
description: 独立收集 Spark UI、YARN、driver、executor 和 eventlog 证据，输出可直接进入最终报告第 2 章的结构化素材。
---

# Spark UI 与日志阅读

## 输入契约

### 必需输入

- `--application_id` 或 `--application_link`

### 可选输入

- `<business_repo_root>/tmp/spark-job-optimization/<case_name>/input/spark_ui/`
- `<business_repo_root>/tmp/spark-job-optimization/<case_name>/input/eventlog/`
- `<business_repo_root>/tmp/spark-job-optimization/<case_name>/input/context/context_report.md`
- `<business_repo_root>/tmp/spark-job-optimization/<case_name>/input/context/cluster.md`
- `<business_repo_root>/tmp/spark-job-optimization/<case_name>/input/context/upstream_tables.md`

### 输入要求

- 两个主参数里只能传一个，优先使用 `--application_id`
- 只允许通过当前机器上的 Chrome 已登录态打开对应的 Spark / YARN 页面并采集执行日志，不尝试其他访问方案
- `--application_link` 必须能在当前机器 Chrome 已登录态中直接打开并定位到 Spark 或 YARN 运行实例
- 如果当前机器 Chrome 已登录态无法打开目标页面，必须返回 `待确认`，不要改用其他方案

## 输出契约

### 必需输出

- 集群与环境信息
- Jobs / Stages / Executors / SQL / EventLog / AM / Driver 证据
- 关键 stage 的耗时、任务数、shuffle、失败、spill
- 运行链路表或流程图
- `已确认` / `待确认`
- 证据缺口

### 输出要求

- 输出可直接进入最终报告第 2 章
- 不能提前进入根因排序
- 不能把推断写成已确认结论
- 默认先落盘到 `<business_repo_root>/tmp/spark-job-optimization/<case_name>/tmp/`，不要写到 skill 仓库或当前工作目录

## 非目标

- 不做根因排序
- 不输出优化建议
- 不改写原始证据为摘要

## 执行流程

1. 用当前机器 Chrome 已登录态打开 `--application_id` 或 `--application_link` 对应页面。
2. 按 `jobs`、`stages`、`executors`、`environment`、`sql` 的顺序查看证据。
3. 对于重要或长时间运行的 job / stage，继续进入 stage 详情页，采集 task 级证据。
4. 采集完证据后，结合 `references/execution-abnormality-rules.md` 判断当前执行是否存在不合理现象。
5. 只记录直接证据，并保持 stage 到源码的对齐关系明确。
6. 停止在根因排序之前，把根因分析留给父级优化 skill。

## 验收标准

- 能独立完成运行证据采集
- 能明确标出 stage 和源码的对齐关系
- 能指出仍需补采的证据缺口

## 参考

`references/README.md`：Spark UI 阅读子 skill 索引。  
`references/evidence-rules.md`：输入约定、证据清单、标签规则、输出要求和分析约束。  
`references/input-parameters.md`：参数说明、优先级和输入组合方式。  
`references/execution-abnormality-rules.md`：执行不合理的定义，以及后续可扩展的场景槽位。  
`references/step-2-contract.md`：第 2 步 Spark UI / 日志阅读调用契约。
