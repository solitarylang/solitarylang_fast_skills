# `source-reading` 资源

这个目录放 `source-reading` 子 skill 专属的样例、模板和结果框架。

## 文件

- `source-reading-stage-sample.md`：源码阅读 stage 划分样例，展示如何从代码行范围切分 stage，并输出稳定的结构。
- `source-reading-result-sample.md`：源码阅读结果样例框架，展示最终输出应包含的固定要素和表格结构。
- `source-reading-result-template.md`：源码阅读结果强模板，规定后续输出必须包含的固定表格和字段。

## 使用要求

- 后续 `source-reading` 输出应优先遵循 `source-reading-result-template.md`。
- 如果需要展示范式，可参考 `source-reading-stage-sample.md` 和 `source-reading-result-sample.md`。
- 输入只需要包含源码目录、主类/主函数、传入参数这些元素，不需要单独的输入模板文件。
- 这里的资源只属于 `source-reading` 子 skill，不应放到主 skill 的 `references/assets/` 下。
