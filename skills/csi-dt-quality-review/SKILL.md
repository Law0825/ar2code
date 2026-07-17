---
name: csi-dt-quality-review
description: 用于独立裁决 DT Case、真实测试代码和实现前基线是否足以约束代码实现时。
disable-model-invocation: true
metadata:
  pattern: dt-quality-review
---

# DT 质量评审

## 目标

基于完整 SDD、DT Case、真实测试资产和本轮基线证据，裁决当前 DT 是否具备代码实现准入条件。实施回流存在时，继承 implementation-review、DT Post 和 code-review 的 finding ID、owner 与证据。

## 评审方法

1. 核对需求/设计锚点到 Case、测试文件、测试名称和断言的追踪；
2. 核对每项自动化 Case 是否被真实测试承接，其他 disposition 是否保持有效理由；
3. 核对测试边界、Oracle、fixture 和 test double 是否保持目标行为与契约；
4. 核对关键 DT 是否启用，测试发现、执行命令和输出是否来自本轮；
5. 逐项核验 `valid_red`、`valid_green`、`invalid_test`、环境阻塞和既有失败的证据；
6. 核对最低测试能力变更是否具有设计来源且不改变生产行为；
7. 确认实现后可复用的稳定测试清单与命令完整。
8. 核对所有直接输入版本一致，稳定测试清单包含文件 hash 与断言锚点。

### Gate 自检

在裁决前，必须显式完成以下覆盖确认：

- 每个自动化 Case 都能追溯到设计锚点、测试文件、测试名称和断言；
- 每个 `valid_red` 都有可解释的失败原因，且失败原因指向目标行为缺口；
- 每个 `valid_green` 都有判别力说明，能解释为什么不是弱测试；
- 每个 `invalid_test` 都归因到具体测试资产问题，而不是泛泛写成“重试”；
- 每个 `blocked` 都说明阻塞事实与责任归属。

若存在任一红牌信号，必须降级为 `conditional_pass` 或 `fail`，不能为了推进流程直接 `pass`。

红牌信号包括：

- 只看到测试通过，没有验证其判别力；
- 只看到失败，没有确认失败是目标缺口还是测试错误；
- mock、stub 或 test-only 接口吞掉了核心行为；
- 关键 Case 没有覆盖到；
- 稳定测试清单缺少 hash 或断言锚点；
- 版本链不一致仍试图给准入。

除以上核对外，对 automated Case 逐项执行以下质量检查：

- **最小性**：一个测试是否只验证一个核心行为或一个紧密耦合的契约命题；
- **清晰性**：测试名称、Case objective 和核心断言能否直接说明意图；
- **判别力**：`valid_red` 是否明确证明“先失败”，`valid_green` 是否明确证明“通过仍有约束力”；
- **真实行为**：断言是否主要观察公开行为，而不是 mock、桩对象或实现细节；
- **mock 合理性**：依赖替代是否位于正确边界，是否错误抹掉了测试依赖的真实副作用；
- **边界覆盖**：错误路径、边界条件和设计要求的异常行为是否有对应验证；
- **输出洁净度**：运行结果是否存在未解释的 warning、error 或脏输出，避免把环境噪音当作通过证据。

发现以下红牌信号时，不论整体命令是否通过，都不得给出 `pass`：

- 新增 Case 未被单独观察其失败或可信 Green；
- 无法解释某个失败属于目标行为缺口还是测试资产错误；
- 测试只验证 mock 行为、辅助对象存在或内部实现细节；
- 为了让测试成立，引入了 test-only 生产接口或隐藏行为开关；
- `valid_green` 缺少判别力说明；
- 实现准入前已经存在“先写实现后补测试”的事实证据。

## 裁决语义

- `pass`：DT 产物均为 `ready`，所有自动化 Case 具有可执行测试，基线只包含有证据的 `valid_red`/`valid_green` 与已隔离的既有失败，可以进入代码实现；
- `conditional_pass`：需求和 SDD 稳定，DT 用例、测试代码、基线证据或最低测试能力存在当前阶段可自动修订的问题；
- `fail`：需求判定、系统契约、test seam 或设计围栏需要上游实现设计修订。

每个问题包含 ID、来源、证据、影响、owner、修复条件和状态。`owner=dt` 对应 `conditional_pass`；`owner=design | requirement` 对应 `fail`。

若问题仅涉及测试粒度、命名、弱 Oracle、mock 边界不当、缺少 Red/Green 证据或测试质量清单未满足，归为 `owner=dt`；若问题指向 test seam 缺失、正式契约不清、围栏或设计边界不足，则继续归为 `owner=design | requirement`。

实施回流中 `owner=dt` 的 finding 在 DT 资产修订后复核；`owner=design | requirement` 的 finding 保持 open 并以 `fail` 交给实现设计，由 SDD 评审继续区分设计修订或需求澄清。

## 产物契约

```yaml
---
ar_id: <AR-ID>
run_id: <runId>
artifact: dt-quality-review
status: blocked | reviewed
verdict: pass | conditional_pass | fail
input_versions:
  design: <updated_at>
  plan: <updated_at>
  task: <updated_at>
  design_review: <updated_at>
  dt_cases: <updated_at>
  test_code: <updated_at>
  baseline: <updated_at>
  implementation_review: <updated_at，存在实施回流时>
  dt_post: <updated_at，存在实施回流时>
  code_review: <updated_at，存在实施回流时>
updated_at: <ISO-8601>
---
```

正文包含追踪结论、测试质量、基线有效性、范围核验、blocking issues、accepted risks、稳定测试清单摘要、裁决理由和下一步类别。评审只维护质量报告。
