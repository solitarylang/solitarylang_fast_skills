# 第 6 步：优化效果验证

## 目的

在代码已经确认并落地后，重新采集优化后 Spark UI / 日志，对比优化前后运行差异，并把代码变更映射到实际执行变化上。

## 输入

- 优化前 Spark UI / 日志快照：`spark_ui/browser/`
- 优化后 Spark UI / 日志快照：`spark_ui/optimized_browser/`
- 优化前 / 优化后代码快照：`source_before/`、`source_after/` 或 `source/`、`source_optimized/`
- 第 5 步确认后的代码变更说明

## 流程

1. 重新采集优化后 Spark UI / 日志。
2. 对比优化前后关键指标：运行时长、job 数、stage 耗时、shuffle、失败数、dead executor、task 数、扫描量。
3. 把优化前后的运行差异和第 5 步代码变更对应起来，形成源码 -> 执行情况映射。
4. 沉淀本次验证出的新经验和可复用规则。

## 输出

- 优化效果报告素材
- 优化前后关键指标对比
- 源码 -> 执行情况映射图
- 推荐文件名：`optimization_result.svg` / `optimization_result.png`
- 效果结论
- 新经验和可复用规则

## 图片要求

- 第 6 步必须输出一张优化结果映射图。
- 这张图要体现优化前执行情况、优化后执行情况和对应的代码变更关系。
- 说明文字必须是中文，代码和命令保留原样。
- 如果文字可能越界，优先增加留白或缩小字号，不允许压线或越框。
- 图片结构优先参考 `references/assets/step6-report-sample.svg` 与 `references/assets/step6-report-sample.png`。
