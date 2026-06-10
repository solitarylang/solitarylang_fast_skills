# 报告产物契约

## 目的

把整个优化流程的输出统一成一组标准产物，避免每个 case 通过改脚本来适配展示。

## 目录基准

所有相对路径都以业务仓库下的 case 工作区为基准：

```text
<business_repo_root>/tmp/spark-job-optimization/<case_name>/
```

其中：

- `input/`：源码快照、Spark UI、eventlog、上下文和原始证据
- `tmp/`：结构化中间结果、草稿 Markdown、渲染中间态
- `output/`：最终报告正文和确认版图片
- `log/`：采集、渲染、执行和调试日志

## 必需字段

### manifest.json

```json
{
  "version": "1.1",
  "case_name": "sample_case",
  "business_repo_root": "<business_repo_root>",
  "case_root": "tmp/spark-job-optimization/sample_case/",
  "input_root": "tmp/spark-job-optimization/sample_case/input/",
  "tmp_root": "tmp/spark-job-optimization/sample_case/tmp/",
  "output_root": "tmp/spark-job-optimization/sample_case/output/",
  "log_root": "log/spark-job-optimization/sample_case/",
  "source": {
    "before": "input/source_before/",
    "after": "input/source_after/"
  },
  "spark_ui": {
    "before": "input/spark_ui/browser/",
    "after": "input/spark_ui/optimized_browser/"
  },
  "report_files": [
    "report/final-report.md",
    "report/step4.md",
    "report/step5.md",
    "report/step6.md"
  ],
  "figure_files": [
    "figures/step4_report.svg",
    "figures/step5_code_change.svg",
    "figures/optimization_result.svg"
  ]
}
```

### analysis.json

```json
{
  "version": "1.0",
  "case_name": "sample_case",
  "stages": [],
  "hotspots": [],
  "evidence_gaps": []
}
```

### hotspots.json

```json
{
  "version": "1.0",
  "case_name": "sample_case",
  "items": [
    {
      "rank": 1,
      "title": "长历史全量扫描",
      "stage": "Stage 1",
      "code_range": "job.py:18-64",
      "evidence": ["6.4 TiB input", "129.4 GiB shuffle write"],
      "state": "已确认",
      "expected_benefit": "高",
      "proposal": "中间表化"
    }
  ]
}
```

### proposals.json

```json
{
  "version": "1.0",
  "case_name": "sample_case",
  "items": []
}
```

### verification.json

```json
{
  "version": "1.0",
  "case_name": "sample_case",
  "before": {},
  "after": {},
  "diff": {},
  "lessons": []
}
```

## 产物生成顺序

1. `analysis.json`
2. `hotspots.json`
3. `proposals.json`
4. `verification.json`
5. `manifest.json`
6. Markdown 报告
7. 图片展示

## 为什么这样更稳

- 数据文件和展示文件解耦
- case 差异只影响数据，不影响渲染器
- 后续增加新规则时，只要扩展 JSON 字段，不需要重写布局逻辑
