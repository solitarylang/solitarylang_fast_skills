# `references/` 索引

这个目录按职责分组，避免把所有说明文件平铺在根目录。

## 目录分工

- `analysis/`：流程、经验规则、判定口径、阶段分析契约
- `input/`：输入目录结构、上下文采集口径
- `report/`：第 1 到第 5 步报告规范、模板和正文要求
- `contracts/`：结构化产物契约和渲染契约
- `assets/`：样例正文、图片和辅助说明
- `subskills/`：独立子技能目录；第 1 / 2 / 6 步仍可保留为独立能力，第 3 / 4 步已并入主 skill 核心流程
- `analysis/step-3-root-cause-ranking.md`：第 3 步根因排序规则，主 skill 内置分析入口
- `analysis/step-4-optimization-proposal.md`：第 4 步优化建议规则，主 skill 内置分析入口
- `report/step-4-report-template.md`：第 4 章正文模板，主 skill 内置编排入口
- `../subskills/verification-and-benchmark/`：优化验证与对比 sub skill

## 维护原则

1. 新增分析规则优先放 `analysis/`
2. 新增报告写作要求优先放 `report/`
3. 新增结构化输出优先放 `contracts/`
4. 新增样例素材优先放 `assets/`
