# 第 1 步源码阅读调用契约

## 输入

### 必需

- `--doc`
  - 源码目录
  - 源码压缩包
  - 描述源码位置、入口文件和主链路的文档

### 可选

- `<business_repo_root>/tmp/spark-job-optimization/<case_name>/input/context/context_report.md`
- `<business_repo_root>/tmp/spark-job-optimization/<case_name>/input/context/upstream_tables.md`
- `<business_repo_root>/tmp/spark-job-optimization/<case_name>/input/spark_ui/browser/`
- `<business_repo_root>/tmp/spark-job-optimization/<case_name>/input/eventlog/`

## 输出

### 必需

- stage 级源码对照表
- `源码 -> stage -> 指标` 表
- 代码路径、算子链、数据流、action 点
- 风险点和观察项
- `已确认` / `待确认`
- 证据缺口

### 输出要求

- 输出可直接进入最终报告第 1 章
- 每个 stage 必须能回指到源码范围
- 不能输出根因排序和优化建议

## 非目标

- 不分析 Spark UI 运行证据
- 不做根因排序
- 不输出优化方案

## 验收

- 输入足以定位至少一个源码入口
- 输出可直接供父 skill 进入第 3 步
- 每个 stage 结果都能回指到源码和缺失证据
