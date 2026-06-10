---
name: pyspark-source-reading
description: 用于通过 --doc 指定源码位置或源码说明文档，独立阅读 PySpark 源码并按 Spark stage 拆分代码路径，输出第 1 章素材。适用于只需要源码侧链路、风险点和 stage 对齐分析的场景。
---

# PySpark 源码阅读

## 执行流程

0. 读取 `--doc`。
   `--doc` 可以直接指向源码目录、源码压缩包，或者一份描述源码位置、入口文件、主链路和上下文的文档。若 `--doc` 是文档，先从文档中提取源码位置和入口，再继续分析。
1. 读取源码入口、主链路和可用上下文。
2. 按 Spark stage 拆分代码路径。
3. 记录每个 stage 的算子链、输入输出、增行点、shuffle 点、冗余列和重复聚合。
4. 标记 `已确认` 与 `待确认`。
5. 产出源码到 stage 的对照结果，并把需要补证据的部分交给日志分析子 skill。

## 参考

`references/source-analysis-rules.md`：源码阅读的识别规则、输出要求和约束。
