---
name: csi-code-implementation
description: 用于根据已通过的 SDD、任务清单和 DT 实现前基线完成生产代码时。
disable-model-invocation: true
metadata:
  pattern: code-implementation
---

# 代码实现

## 目标

按 `design.md` 的目标与围栏、`plan.md` 的依赖顺序、`task.md` 的变更单元和已通过的 DT 基线完成生产实现。

## 执行方法

1. 确认输入版本和 DT 质量评审为 `pass`；
2. 按 plan 波次和 task 依赖选择当前可执行任务；
3. 从任务声明的路径、符号和相邻实现模式读取必要代码；
4. 实现满足设计契约和任务完成条件的最小变更；
5. 运行与本次编辑直接相关的编译、静态检查或快速检查；
6. 更新任务映射、实际变更和验证证据；
7. 逐个完成剩余任务，保持接口、迁移、兼容和回滚顺序。

实现优先复用现有模块、错误处理、配置和数据模式。新增文件或抽象记录其设计锚点和必要性。生产代码、运行配置、迁移与正式文档由 task 围栏授权；基线测试文件与断言保持原内容 hash，测试资产修订通过 DT 回流重新建立基线。

当实现发现测试资产与正式契约不一致时，在摘要中记录 `owner=dt`、证据和影响；当实现路径要求改变正式契约或围栏时记录 `owner=design`。对应职责在状态回流后修订。

## 产物契约

`implementation-summary.md` 使用：

```yaml
---
ar_id: <AR-ID>
run_id: <runId>
artifact: implementation-summary
status: draft | blocked | ready
design_version: <design.md updated_at>
task_version: <task.md updated_at>
dt_quality_version: <dt-quality-review.md updated_at>
updated_at: <ISO-8601>
---
```

正文包含任务完成矩阵、Changed Files、关键实现决策、迁移/兼容/回滚处理、快速检查命令与结果、未决问题和下一步 DT 执行清单。所有 task 已完成且变更可交给实现后 DT 执行时设置为 `ready`。
