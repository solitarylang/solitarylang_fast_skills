---
name: spark-job-optimization
description: 用于通过源码和 Spark UI 日志分析 Spark / PySpark 任务，定位瓶颈并按确认后的顺序执行最小化优化。
---

# Spark 作业优化

## 执行流程

1. 采集集群资源和上游表上下文。
2. 调用源码分析子 skill，形成最终报告第 1 章素材。
3. 调用 Spark UI / 日志分析子 skill，形成最终报告第 2 章素材。
4. 使用主 skill 内置的根因分析与优化建议能力，结合第 1 / 2 章素材形成最终报告第 3 章和第 4 章素材。
5. 根据用户确认执行代码变更，形成最终报告第 5 章素材，并编排最终报告正文。
6. 调用优化验证与对比子 skill，形成最终报告第 6 章素材。

## 组织原则

- 第 3 / 4 步是主 skill 的核心分析能力，不再作为独立 sub skill 对外暴露
- 第 1 / 2 / 6 步仍可按子技能方式独立维护，便于单步调试与复用
- 主 skill 负责跨步骤的串联、总控和最终汇总，同时承载根因排序、优化建议和报告编排的核心逻辑
- 第 3 / 4 步的规则、模板和样例统一维护在 `references/analysis/` 和 `references/report/` 下，作为主流程的 canonical 入口

## 落盘原则

- skill 可以在任意工作目录调用，不能默认当前目录就是业务仓库根。
- 执行前必须先确定 `business_repo_root`，所有生成内容都写到业务仓库下。
- 统一工作根目录为：

```text
<business_repo_root>/tmp/spark-job-optimization/<case_name>/
<business_repo_root>/log/spark-job-optimization/<case_name>/
```

- `input/`：原始输入、源码快照、Spark UI、eventlog、上下文采集结果。
- `tmp/`：中间 JSON、草稿 Markdown、渲染中间态、临时对比结果。
- `output/`：最终报告正文、图片和确认版产物。
- `log/`：采集日志、渲染日志、执行日志、调试日志。
- 不要把中间结果散落在 skill 仓库或当前 shell 目录下。

## 参考

`references/README.md`：`references/` 目录总索引。  
`references/analysis/README.md`：分析规则和经验口径索引。  
`references/input/README.md`：输入契约和上下文采集索引。  
`references/report/README.md`：报告规范和正文模板索引。  
`references/contracts/README.md`：输出契约和渲染契约索引。  
`references/assets/README.md`：正文样例、图片样例和端到端演示样例索引。
