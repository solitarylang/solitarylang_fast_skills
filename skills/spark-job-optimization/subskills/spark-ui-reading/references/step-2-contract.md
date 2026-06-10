# 第 2 步 Spark UI / 日志阅读调用契约

## 输入

### 必需

- `--application_id` 或 `--application_link`

### 可选

- `<business_repo_root>/tmp/spark-job-optimization/<case_name>/input/spark_ui/`
- `<business_repo_root>/tmp/spark-job-optimization/<case_name>/input/eventlog/`
- `<business_repo_root>/tmp/spark-job-optimization/<case_name>/input/context/context_report.md`
- `<business_repo_root>/tmp/spark-job-optimization/<case_name>/input/context/cluster.md`
- `<business_repo_root>/tmp/spark-job-optimization/<case_name>/input/context/upstream_tables.md`

## 输出

### 必需

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

## 访问约束

- 只允许通过当前机器上的 Chrome 已登录态打开对应的 Spark / YARN 页面并采集执行日志
- 不尝试其他访问方案
- 如果 Chrome 已登录态无法打开目标页面，只能返回 `待确认`

## 非目标

- 不做根因排序
- 不输出优化建议
- 不改写原始证据为摘要

## 验收

- 能独立完成运行证据采集
- 能明确标出 stage 和源码的对齐关系
- 能指出仍需补采的证据缺口
