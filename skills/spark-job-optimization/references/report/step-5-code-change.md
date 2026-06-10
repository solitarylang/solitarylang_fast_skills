# 第 5 步：代码变更与确认版报告

## 目的

只执行用户已经批准的优化，并产出确认版报告与源码 -> 变更映射图，供后续第 6 步验证使用。

## 输入

- Approved optimization items
- Current source files
- 第 4 步报告与图片
- 人工 review 结论（哪些点能改、哪些点怎么改）

## 流程

1. 先保留优化前代码快照，不依赖 git 作为版本记录。
2. 只改最小必要的代码面。
3. 除非优化明确要求改变行为，否则保持行为一致。
4. 保持可读、可审查。
5. 把修改后的路径再和原瓶颈核一遍。
6. 如果修复会碰到多个热点，只有在它们共享同一个根因时才合并处理。
7. 完成后保留优化后代码快照，确保 `<business_repo_root>/tmp/spark-job-optimization/<case_name>/input/source/` / `.../input/source_before/` 和 `.../input/source_optimized/` / `.../input/source_after/` 都能回溯。
8. 生成源码 -> 代码变更映射图，图里保留原始瓶颈位置、人工确认后的修改点和对应的修改后代码草图。

## 输出

- 更新后的代码
- 简短变更摘要
- 仍未解决的问题（如有）
- 修改路径的验证说明
- 优化确认版报告素材
- 源码 -> 代码变更映射图
- 推荐落盘位置：`<business_repo_root>/tmp/spark-job-optimization/<case_name>/output/figures/step5_code_change.svg` / `.../step5_code_change.png`
- 优化前 / 优化后代码快照位置说明
- 为第 6 步准备的优化后快照和验证入口说明

## 代码快照要求

- 如果还没有优化前快照，先把当前工作副本复制到 `<business_repo_root>/tmp/spark-job-optimization/<case_name>/input/source_before/` 或 `.../input/source/` 的稳定备份目录。
- 修改完成后，再把最新代码复制到 `.../input/source_optimized/` 或 `.../input/source_after/`。
- 这两个目录必须同时保留，作为后续对比的唯一代码版本来源。
- 不要用 git 历史代替代码快照目录，也不要只保留单个版本。

## 图片要求

- 第 5 步必须输出一张确认版映射图。
- 这张图要体现源码、人工确认后的变更点和修改后代码草图的对应关系。
- 说明文字必须是中文，代码和命令保留原样。
- 如果文字可能越界，优先增加留白或缩小字号，不允许压线或越框。
- 图片结构优先参考 `../assets/step5-report-sample.svg` 与 `../assets/step5-report-sample.png`。
