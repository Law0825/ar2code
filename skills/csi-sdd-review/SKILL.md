---
name: csi-sdd-review
description: 用于独立裁决 design、plan 和 task 是否构成可进入 DT 设计的完整 SDD 时。
disable-model-invocation: true
metadata:
  pattern: sdd-review
---

# SDD 统一评审

## 目标

基于正式需求、三份 SDD 产物、挑战 findings 和必要代码证据，裁决当前 SDD 是否具备 DT 设计准入条件。

## 输入处理

使用 Workflow 提供的正式输入，确认 `design.md`、`plan.md` 和 `task.md` 的版本链一致，并核对 design 记录的 clarification 与 clarification-review 直接输入版本。DT 回流存在时，按原 finding ID、证据和 owner 检查 disposition。

## 评审方法

1. 核对需求锚点到 design、plan、task 的追踪；
2. 核对目标架构、契约、围栏和任务动作的一致性；
3. 核对任务落点、依赖、波次、关键路径和完成条件；
4. 核对迁移、兼容、回滚和验证顺序；
5. 核对挑战 findings 和 upstream gaps 的处理状态；
6. 确认 task 为 DT 提供验证义务、入口和 test seam。

### Gate 与回流规则

SDD 评审不是确认“看起来合理”，而是确认三层产物可以作为下一阶段的正式输入：

- `design.md`、`plan.md`、`task.md` 必须版本链一致；
- 围栏、依赖方向和任务动作必须互相匹配；
- DT 所需验证义务、入口和 test seam 必须明确存在；
- 若发现 upstream gap，必须回流到对应职责，不得在 SDD 评审内强行放行。

门禁规则：

- 单轮评审超过 2 次仍未收敛时，暂停并请求用户决策；
- 多个产物同时存在问题时，按最坏结果原则判定；
- `conditional_pass` 只允许用于当前阶段可修订的文档问题，不允许掩盖需求失效或围栏断裂；
- `fail` 用于需要重新澄清需求、重做设计边界或修正规格来源的问题。

红牌信号：

- 设计、计划、任务三份文档版本链断裂；
- 围栏在 plan / task 中不一致；
- DT 入口或验证义务不完整；
- challenge findings 还挂着，但试图以“影响不大”直接通过；
- 只有文档存在，没有可执行落点或 test seam。

## 通过条件

只有以下条件全部满足时才能 `pass`：

- `design.md`、`plan.md` 和 `task.md` 均为 `ready`；
- 三份产物的版本链和追踪矩阵一致；
- 代码落点有证据或明确的新建定位策略；
- 依赖、波次和关键路径可执行；
- 设计围栏在 plan 和 task 中保持一致；
- critical 和 major findings 已解决；
- 没有未决 upstream gap 或 DT 回流 finding；
- DT 所需验证义务和入口完整。

## 裁决语义

- `pass`：完整 SDD 可以进入 DT 设计；
- `conditional_pass`：需求基线稳定，design、plan、task 或 finding 处理仍需修订；
- `fail`：存在需要重新澄清的需求事实、需求被改变或正式需求输入失效。

## 产物契约

Workflow 指定的 `design-review.md` 使用：

```yaml
---
ar_id: <AR-ID>
run_id: <runId>
artifact: sdd-review
status: blocked | reviewed
verdict: pass | conditional_pass | fail
input_versions:
  clarification_review: <updated_at>
  design: <updated_at>
  plan: <updated_at>
  task: <updated_at>
  design_challenge: <updated_at>
  dt_quality_review: <updated_at，存在 DT 回流时>
updated_at: <ISO-8601>
---
```

正文包含输入版本、三层追踪结论、五维挑战处理、blocking issues、accepted risks、DT 准入摘要、裁决理由和下一步类别。
