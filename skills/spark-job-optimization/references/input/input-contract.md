# 输入约定

## 业务仓库目录结构

每个分析 case 都落在业务仓库下面的固定工作区，不能假设当前目录就是仓库根。

```text
<business_repo_root>/
  tmp/
    spark-job-optimization/
      <case_name>/
        input/
          source.zip
          source/
          spark_ui/
          context/
          eventlog/
          notes.md
        tmp/
        output/
  log/
    spark-job-optimization/
      <case_name>/
```

## 源码

- 首选：`<business_repo_root>/tmp/spark-job-optimization/<case_name>/input/source.zip`
- 也可以直接放解压后的 `.../input/source/`
- 两者都存在时，把 `source/` 当工作副本，`source.zip` 作为原始证据保留
- 如果有多个源码压缩包，在 `notes.md` 里简短说明哪个是主包
- 为了保留优化前 / 优化后两个版本的代码，不要依赖 git 记录；请直接保留代码快照目录。
- 默认把优化前代码放在 `input/source/`（或 `input/source_before/`），优化后代码放在 `input/source_optimized/`（或 `input/source_after/`）。
- 如果同一个 case 有多个优化版本，优先保留“优化前 + 最新优化后”两个版本，其他历史版本需要在 `notes.md` 里说明语义。

## Spark UI

把导出的 Spark UI 页面放到 `input/spark_ui/`：

- `jobs.html`
- `stages.html`
- `sql.html`
- `executors.html`
- `environment.html`

如果是从登录态浏览器里采集的，可把复制出的页面内容放到：

- `input/spark_ui/browser/environment.md`
- `input/spark_ui/browser/jobs.md`
- `input/spark_ui/browser/stages.md`
- `input/spark_ui/browser/executors.md`
- `input/spark_ui/browser/sql.md`
- `input/spark_ui/browser/details/` 下的重要 job / stage 详情页 Markdown
- `input/spark_ui/browser/manifest.md`

同一个 case 在分析阶段重复采集时，默认覆盖 `input/spark_ui/browser/` 这份当前快照，不要每次都新增并行目录。
如果要保留最终优化后的对比版本，建议另存到单独目录，例如 `input/spark_ui/optimized_browser/`，并在 `notes.md` 里说明它是优化后快照。
分析前的 Spark UI 视为“优化前快照”，优化完成后重新采集的 Spark UI 视为“优化后快照”，两者都要保留，方便做前后对比。
浏览器采集时，表格块要尽量保留成 Markdown table，不要只保存为纯文本；如果某个 job / stage 明显重要或运行超过 30 分钟，要补抓对应详情页和 task 级信息。task 数量过多时，只保留排序后的第一页 task 明细，不需要全量 task 页面。

如果导出的文件名不同，就保留原名，并在 `notes.md` 里说明每个文件是什么。

## 上下文

把采集到的集群资源、上游表备注和兜底查询放到 `input/context/`。

Suggested files:

- `context_report.md`
- `cluster.md`
- `upstream_tables.md`
- `table_queries.md`

## Event Log 和补充证据

- event log 放到 `input/eventlog/`
- 截图放到 `input/spark_ui/` 或 `notes/`
- 运行备注、假设、已知限制写到 `notes.md`

## 输出

结果写到：

```text
<business_repo_root>/tmp/spark-job-optimization/<case_name>/output/
```

输入和输出要保持同一个 case 名，方便端到端追踪。

## 前后对比约定

- `input/spark_ui/browser/` 默认表示优化前当前快照。
- `input/spark_ui/optimized_browser/` 默认表示优化后快照。
- 如果同一个 case 反复优化，保持这两个目录分别代表“优化前 / 优化后”。
- 如果需要保留更早的历史版本，可以另起目录，但必须在 `notes.md` 里说明版本语义。

## 代码版本约定

- `input/source/` 或 `input/source_before/` 默认表示优化前代码快照。
- `input/source_optimized/` 或 `input/source_after/` 默认表示优化后代码快照。
- 不使用 git 来承载版本对比；需要保留的版本请直接落盘成目录。
- 最终优化前后对比时，必须同时保留这两个代码快照，避免后续变更覆盖掉原始版本。
