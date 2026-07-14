---
name: csi-implementation-review
description: 用于统一裁决 AR、SDD、DT、代码变更和实现后 Green 证据是否形成完整实施闭环时。
disable-model-invocation: true
metadata:
  pattern: implementation-review
---

# 实施评审

## 目标

核验从 AR 到实现结果的端到端追踪和证据闭环，裁决工作流是否可以进入只读完成状态。代码审查负责逐文件 findings，本评审只核验 findings 处理和整体准入。

## 评审方法

1. 核对正式输入版本与状态；
2. 核对 AR/澄清 → design → plan/task → DT Case → 测试代码 → 基线 → 代码变更 → Green 的追踪；
3. 核对 task 完成矩阵、Changed Files 和设计围栏；
4. 核对实现后执行覆盖基线稳定测试清单，必要增量具有来源；
5. 核对 `test_status: green`、关键 DT 启用和相关回归结果；
6. 核对基线测试文件 hash、Case 映射与核心断言一致；测试资产变更已经过 DT 重新基线和质量评审；
7. 核对 blocking/major code review findings 的处理状态；
8. 核对迁移、兼容、回滚和正式验收义务已经落实。

## 裁决语义

- `pass`：任务完成、目标 DT 与必要回归为 Green、变更符合围栏、blocking/major findings 已解决、端到端追踪完整；
- `conditional_pass`：需求、SDD 和 DT 基线稳定，编码实现、执行证据或实现类 finding 存在当前代码实现阶段可自动修订的问题；
- `fail`：测试资产、系统设计或需求基线需要上游职责修订。

每个未关闭问题包含 ID、来源、证据、owner、修复条件和阻塞影响。`owner=implementation` 对应 `conditional_pass`；`owner=dt | design | requirement` 对应 `fail`。

`owner=environment` 对应 `conditional_pass`，保留恢复条件和复现命令，由代码实现状态自动重试；状态的 `maxSelfTransitions` 构成有限重试预算，预算耗尽后 ACEHarness 将本次运行标记为失败。

## 产物契约

```yaml
---
ar_id: <AR-ID>
run_id: <runId>
artifact: implementation-review
status: blocked | reviewed
verdict: pass | conditional_pass | fail
input_versions:
  clarification: <updated_at>
  design: <updated_at>
  plan: <updated_at>
  task: <updated_at>
  design_review: <updated_at>
  dt_cases: <updated_at>
  dt_test_code: <updated_at>
  dt_baseline: <updated_at>
  dt_quality: <updated_at>
  implementation: <updated_at>
  dt_post: <updated_at>
  code_review: <updated_at>
updated_at: <ISO-8601>
---
```

正文包含版本核验、端到端追踪、任务与围栏、Green 证据、findings 处理、blocking issues、accepted risks、裁决理由和下一步类别。完成报告后在步骤结果中输出 ACEHarness 标准 verdict JSON。
