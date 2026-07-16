---
name: csi-code-review
description: 用于基于真实 diff 审查实现是否满足 SDD、任务围栏、代码质量和 DT 约束时。
disable-model-invocation: true
metadata:
  pattern: code-review
---

# 代码审查

## 目标

独立检查真实代码变更的正确性、范围、安全性、性能、可维护性和测试完整性，输出有证据的 findings。代码审查维护 findings，实施评审负责准入裁决。

## 审查范围

### 1. 围栏合规性检查（高优先级）
优先使用 `git status --short`、`git diff --stat` 和相关 `git diff` 确认真实变更。然后执行围栏合规性检查：

**围栏检查步骤**：
1. 从 `design.md` 读取"模块变更计划"中的围栏配置
2. 对比实际变更文件与围栏配置
3. 检查所有变更是否在允许范围内

**围栏合规性检查要点**：
- ✅ 允许修改的文件：确认变更符合预期
- ❌ 禁止修改的文件：检查是否有越界修改（如果有，标记为 blocking finding）
- 🔵 条件修改的文件：确认修改满足条件
- 📦 新增的文件：确认有正当的设计原因
- 未在围栏中明确的文件：评估是否应该在禁止范围内

### 2. 详细变更检查
对每个变更文件检查：

- 对应 task、设计锚点和变更围栏；
- 主路径、错误路径、边界、状态和兼容行为；
- 与改动相关的权限、输入信任边界、敏感数据和资源释放；
- 与改动相关的并发、复杂度、I/O 和数据访问风险；
- 项目既有结构、命名、错误处理和可维护模式；
- DT Case、测试代码和 Green 报告是否真实覆盖相关行为；
- 新增文件、抽象和依赖是否具有任务来源与维护价值。

### 3. 围栏突破记录检查
如果有围栏突破，检查：
- 是否有明确的突破原因记录
- 是否有用户授权记录
- 突破原因是否合理

审查深度服从风险与改动规模。修订轮次以最新 diff 和未关闭 finding 为主，复核修复是否引入新问题。

## Finding 契约

每项 finding 包含：

- `id`；
- `severity`: blocking | major | minor；
- `dimension`: correctness | boundary | security | performance | maintainability | testing；
- 文件与符号位置；
- claim、evidence、impact、recommendation；
- `owner`: implementation | dt | design | requirement；
- `resolution_status`: open | resolved。

## 产物契约

```yaml
---
ar_id: <AR-ID>
run_id: <runId>
artifact: code-review
status: reviewed
input_versions:
  design: <design.md updated_at>
  task: <task.md updated_at>
  dt_quality: <dt-quality-review.md updated_at>
  implementation: <implementation-summary.md updated_at>
  dt_post: <dt-test-report-post.md updated_at>
findings_status: open | resolved
updated_at: <ISO-8601>
---
```

正文包含 diff evidence、Changed Files 围栏矩阵、六维结论、findings、已解决项和证据限制。没有问题时明确记录未发现有证据的问题。
