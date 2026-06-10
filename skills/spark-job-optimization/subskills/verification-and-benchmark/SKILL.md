---
name: spark-verification-and-benchmark
description: 独立完成优化前后对比采集、指标验证和经验沉淀，覆盖最终报告第 6 章素材。
---

# 优化验证与对比

## 输入契约

### 必需输入

- 优化前 Spark UI / 日志快照
- 优化后 Spark UI / 日志快照
- 已确认的代码变更说明

### 可选输入

- 优化前 / 优化后代码快照
- 第 5 步确认后的映射表
- 运行备注和补充说明

### 输入要求

- 必须同时具备 before 和 after 两份快照
- 如果缺少任一侧快照，只能输出 `待确认`
- 输入需要能定位到变更前后对应的 stage 和代码范围

## 输出契约

### 必需输出

- 优化前后关键指标对比
- 运行时长、job 数、stage 耗时、shuffle、失败数、dead executor、task 数、扫描量对比
- 源码 -> 执行情况映射表
- 源码 -> 执行情况映射图所需的摘要
- 效果结论
- 新经验和可复用规则

### 输出要求

- 可直接进入最终报告第 6 章
- 必须明确对比基线和验证结果
- 不能把推断写成已确认结论
- 默认先落盘到 `<business_repo_root>/tmp/spark-job-optimization/<case_name>/tmp/`，不要写到 skill 仓库或当前工作目录

## 非目标

- 不做代码变更
- 不重新做根因排序
- 不替代第 3/4 步的优化建议

## 执行流程

1. 读取优化前后 Spark UI / 日志快照。
2. 对比运行时长、job 数、stage 耗时、shuffle、失败数、dead executor、task 数和扫描量。
3. 把运行变化映射回已确认的代码变更。
4. 沉淀可复用经验和新规则。

## 验收标准

- 能独立完成 before/after 对比
- 能明确指出优化带来的实际变化
- 输出可直接进入最终报告第 6 章

## 参考

`references/README.md`：验证 sub skill 索引。  
`references/step-6-final-verification.md`：优化效果验证规则。  
