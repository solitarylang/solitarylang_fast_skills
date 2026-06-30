# 分析规则索引

这个目录放流程和经验规则，目标是统一管理“怎么分析、怎么判断、怎么排序”。

## 文件

- `optimization-rules.md`：输入约定、输出原则、分析规则和写作约束
- `diagnostic-heuristics.md`：可复用的诊断启发式和排序经验
- `step-3-root-cause-ranking.md`：第 3 步根因排序规则，属于主 skill 核心分析能力
- `step-4-optimization-proposal.md`：第 4 步优化建议规则，属于主 skill 核心分析能力
- `spark_ui_label_definition.sql`：Spark UI 标签定义样例
- `../../subskills/source-reading/references/step-1-contract.md`：第 1 步源码阅读调用契约
- `../../subskills/spark-ui-reading/references/step-2-contract.md`：第 2 步 Spark UI / 日志阅读调用契约
- `../../subskills/verification-and-benchmark/references/step-6-final-verification.md`：第 6 步优化验证规则

## 使用方式

如果新增的是“分析方法、判定口径、排序经验、标签定义”，都优先放到这里。
如果是第 3 / 4 步的根因排序、优化建议和模板规则，也直接放到这里或 `report/` 下，不再以独立 sub skill 方式维护。
