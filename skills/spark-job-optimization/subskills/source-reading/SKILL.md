---
name: pyspark-source-reading
description: 独立阅读 PySpark 源码并按 Spark stage 拆分代码路径，输出可直接进入最终报告第 1 章的结构化素材。
---

# PySpark 源码阅读

## 输入契约

### 必需输入

- `--doc`
  - 源码目录
  - 源码压缩包
  - 描述源码位置、入口文件、主链路和上下文的文档

### 可选输入

- `<business_repo_root>/tmp/spark-job-optimization/<case_name>/input/context/context_report.md`
- `<business_repo_root>/tmp/spark-job-optimization/<case_name>/input/context/upstream_tables.md`
- `<business_repo_root>/tmp/spark-job-optimization/<case_name>/input/spark_ui/browser/` 或 `<business_repo_root>/tmp/spark-job-optimization/<case_name>/input/eventlog/`

### 输入要求

- `--doc` 必须能定位到至少一个源码入口
- 如果 `--doc` 是文档，先提取源码位置和入口，再继续分析
- 如果输入不足以完成 stage 拆分，必须明确返回 `待确认`

## 输出契约

### 必需输出

- stage 级源码对照表
- `源码 -> stage -> 指标` 表
- 代码路径、算子链、数据流、action 点
- 风险点和观察项
- `已确认` / `待确认`
- 需要第 2 步补齐的证据缺口

### 输出要求

- 输出可直接进入最终报告第 1 章
- 每个 stage 必须回指到具体代码范围
- 不能输出根因排序和优化建议
- 默认先落盘到 `<business_repo_root>/tmp/spark-job-optimization/<case_name>/tmp/`，不要写到 skill 仓库或当前工作目录

## 非目标

- 不分析 Spark UI 运行证据
- 不做根因排序
- 不输出优化方案

## 执行流程

1. 读取 `--doc` 和可选上下文。
2. 识别入口文件、主链路和数据流。
3. 按 Spark stage 拆分代码路径。
4. 记录每个 stage 的算子链、输入输出、增行点、shuffle 点、冗余列和重复聚合。
5. 标记 `已确认` 与 `待确认`。
6. 产出源码到 stage 的对照结果，并把需要补证据的部分交给日志分析子 skill。

## 验收标准

- 能独立完成源码侧 stage 拆分
- 能输出可直接进入第 1 章的内容
- 能清晰指出还需要日志补证据的地方

## 参考

`references/README.md`：源码阅读子 skill 索引。  
`references/source-analysis-rules.md`：源码阅读的识别规则、输出要求和约束。  
`references/step-1-contract.md`：第 1 步源码阅读调用契约。
