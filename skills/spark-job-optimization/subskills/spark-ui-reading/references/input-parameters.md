# 输入参数说明

## 目标

把 Spark UI / 日志阅读子 skill 的输入方式标准化，避免只靠口头约定。

## 推荐参数

- `--application_id`：YARN / Spark 应用 id，优先使用。
- `--application_link`：应用链接，优先支持 YARN application 链接，也可用于 Spark UI 链接。

## 参数优先级

1. 先用 `--application_id` 锁定运行实例。
2. 如果没有应用 id，则用 `--application_link` 作为入口。

## 典型组合

- `--application_id`
- `--application_link`
- `--application_id` + `--application_link`（以 `--application_id` 为准）

## 约定

- 只允许一个主参数。
- 如果同时传入多个来源，以 `--application_id` 为准。
- 进入证据采集时，只允许通过当前机器上的 Chrome 已登录态访问对应页面，不尝试其他方案。
- 如果 Chrome 已登录态无法打开目标页面，直接返回 `待确认`，不要切换浏览器、重登账号或改用其他访问方式。
