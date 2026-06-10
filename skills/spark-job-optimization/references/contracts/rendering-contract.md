# 渲染契约

## 目标

把最终展示从“按 case 改脚本”改成“按固定契约输出数据，再由通用渲染器生成展示物”。

## 核心原则

1. **渲染器必须是通用的**
   - 不允许把某个 case 的热点、行号、证据或结论写死在渲染代码里。
   - 新增 case 时，只能补充数据文件，不能要求修改渲染逻辑。

2. **展示和分析分离**
   - 分析层输出结构化数据。
   - 渲染层只负责排版、对齐和样式。

3. **数据优先**
   - 所有结论、指标、证据和待确认项先落到 JSON / Markdown / YAML。
   - 图和最终报告只是这些数据的视图。

4. **版本可演进**
   - schema 要带 `version` 字段。
   - 渲染器对未知字段应忽略，不因新增字段而失效。

## 推荐输出结构

```text
<business_repo_root>/tmp/spark-job-optimization/<case_name>/
  input/
    source/
    spark_ui/
    context/
    eventlog/
  tmp/
    analysis.json
    hotspots.json
    proposals.json
    verification.json
    manifest.json
  output/
    report/
      final-report.md
      step4.md
      step5.md
      step6.md
    figures/
      step4_report.svg
      step4_report.png
      step5_code_change.svg
      step5_code_change.png
      optimization_result.svg
      optimization_result.png
  log/
    collect.log
    render.log
    verify.log
```

## 推荐数据流

1. 源码分析产出 `analysis.json`
2. Spark UI / 日志分析产出 `evidence.json`
3. 根因排序产出 `hotspots.json`
4. 优化方案产出 `proposals.json`
5. 人工确认后产出 `changes.json`
6. 优化验证产出 `verification.json`
7. 通用渲染器读取这些 JSON，生成 Markdown 和图片

## 什么叫“通用渲染器”

渲染器只应该做这些事：

- 读取标准化数据
- 套用固定模板
- 生成 Markdown / SVG / PNG
- 保证版式一致性

渲染器不应该做这些事：

- 为某个 case 写专属判断逻辑
- 把某个热点直接 hardcode 进代码
- 根据业务语义临时改版式

## 最稳妥的落地方式

如果不希望维护复杂绘图脚本，优先采用：

1. **Markdown 作为主输出**
   - 第 1 到第 6 步都先生成 Markdown。
   - 方便人工 review，也方便后续回放。

2. **单一模板渲染**
   - 只保留一个固定模板渲染器。
   - 输入是标准 JSON，不是脚本参数拼接。

3. **图片只做“最终视图”**
   - 图片由通用模板生成。
   - 若某个 case 没有足够证据，可以自动降级为更简洁布局，而不是改脚本。

## 自动降级规则

如果某一类数据缺失，渲染器应自动降级：

- 没有 task 级证据 -> 卡片显示 `待确认`
- 没有 SQL plan -> 只显示 SQL 原文不可用
- 没有优化后快照 -> 第 6 步仅输出文字版，不强制出图
- 热点数量少于 5 个 -> 只渲染实际存在的热点，不补空位

## 最终判断标准

如果后续只需要改：

- schema 文档
- 分析规则文档
- case 数据文件

而不需要改渲染器代码，就说明这套展示已经足够健壮。
