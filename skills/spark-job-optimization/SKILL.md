---
name: spark-job-optimization
description: 用于通过源码和 Spark UI 日志分析 Spark / PySpark 任务，定位瓶颈并按确认后的顺序执行最小化优化。
---

# Spark 作业优化

## 执行流程

1. 采集集群资源和上游表上下文。
2. 调用源码分析子 skill，形成最终报告第 1 章素材。
3. 调用 Spark UI / 日志分析子 skill，形成最终报告第 2 章素材。
4. 结合代码路径和运行证据定位慢点，形成最终报告第 3 章素材。
5. 按影响度排序前 5 个有证据支撑的瓶颈，给出优化建议、预期收益和代码草图，并等待用户确认。
6. 根据用户确认执行代码变更，形成最终报告第 5 章素材。
7. 重新采集优化后 Spark UI / 日志，并与优化前快照对比，形成最终报告第 6 章素材。

## 参考

`references/optimization-rules.md`：输入约定、输出原则、分析规则和写作约束。  
`references/context-collection.md`：集群资源和上游表上下文的采集口径。  
`references/step-1-source-reading.md`：源码分析章节的写作要求。  
`references/step-2-spark-ui-reading.md`：日志分析章节的写作要求。  
`references/step-3-root-cause-ranking.md`：根因排序章节的写作要求。  
`references/step-4-optimization-proposal.md`：优化方案章节的写作要求。  
`references/step-4-report-template.md`：第 4 章报告模板。  
`references/step-5-code-change.md`：优化代码变更章节的写作要求。  
`references/step-6-final-verification.md`：优化效果验证章节的写作要求。
