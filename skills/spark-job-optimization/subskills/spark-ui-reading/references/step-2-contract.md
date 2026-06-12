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
- 要补充 Spark 运行参数解读：说明参数和当前日志 / Spark UI 是否一致；如果不一致，记录为可验证的调参候选
- 单页采集最多重试 3 次；如果仍然没采到有效内容，立即终止本次采集任务，不再继续调试或反复重试
- 每个采集文件必须和页面一一对应，页面 URL、文件名、失败状态要在 `manifest.md` / `manifest.json` 中同步记录
- 如果某页打开错误或没有采到有效内容，不要把别页内容写进这个文件，改为保留对应的 `mismatch` 失败证据

## 访问约束

- 只允许通过当前机器上的 Chrome 已登录态打开对应的 Spark / YARN 页面并采集执行日志
- 不尝试其他访问方案
- 如果 Chrome 已登录态无法打开目标页面，只能返回 `待确认`

## 非目标

- 不做根因排序
- 不输出脱离证据的优化建议；如果发现明显参数失配，只记录可验证的调参候选
- 不改写原始证据为摘要

## 验收

- 能独立完成运行证据采集
- 能明确标出 stage 和源码的对齐关系
- 能指出仍需补采的证据缺口
