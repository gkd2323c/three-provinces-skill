# Plan Object 格式规范

三省之间传递信息的结构化契约。使用 YAML 格式。
文件层级按「三省流转」划分为三大区块：`plan` → `review` → `results`，各区块由对应角色独立填充。

---

## 完整结构

```yaml
plan_id: plan_YYYYMMDD_HHMMSS
version: 1
original_task: "<用户的原始请求全文>"
created_by: "zhongshusheng"

context:
  complexity:
    score: 14
    breakdown:
      steps: 4
      domains: 2
      uncertainty: 3
      failure_cost: 2
      tool_chain: 1
  domains:
    - "<涉及的领域1>"
    - "<领域2>"
  known_context: "<尚书省提供的已知上下文全文>"

# ── 中书省填充 ──
plan:
  goal: "<一句话描述本次任务的目标>"
  revision: 1
  subtasks:
    - id: sub-01
      summary: "<简短标题，10字以内>"
      department: "<归属部门>"
      prompt: "<给执行者的完整指令，包含做什么、怎么做、注意什么>"
      tools_hint:
        - "<建议使用的工具>"
      depends_on: []
      expected_output: "<执行完成后应产出的具体成果>"
    - id: sub-02
      summary: "..."
      department: "<归属部门>"
      prompt: "..."
      tools_hint:
        - "..."
      depends_on:
        - "sub-01"
      expected_output: "..."

risks:
  - "<这项任务可能在什么地方遇到困难>"
  - "<需要额外注意的事项>"

# ── 门下省填充 ──
review:
  result: approved          # approved | rejected
  round: 1
  checklist:
    completeness: true
    intent_alignment: true
    granularity_and_structure: true
    dependency_correctness: true
    prompt_quality: true
    safety: true
    tool_accessibility: true
  rejection_reason: "<打回理由，仅在 rejected 时有值>"
  suggestion: "<改进建议，仅在 rejected 时有值>"
  notes: "<补充意见，可选>"

# ── 尚书省填充 ──
results:
  status: done              # done | partial | blocked
  subtasks:
    - id: sub-01
      output:
        status: done        # done | blocked | skipped
        summary: "<一句话总结做了什么>"
        artifacts:
          - path: "<产出文件路径>"
            description: "<文件说明>"
        notes: "<执行中发现的注意事项>"
    - id: sub-02
      output:
        status: done
        summary: "..."
        artifacts: []
        notes: ""
  summary: "<尚书省汇总后的回奏全文>"
```

---

## 字段说明

### 顶层字段

| 字段 | 填充者 | 说明 |
|------|--------|------|
| `plan_id` | 尚书省 | 格式 `plan_YYYYMMDD_HHMMSS`，同一 session 内唯一 |
| `version` | 尚书省 | 当前 Plan Object 格式版本，当前为 `1` |
| `original_task` | 尚书省 | 用户的原始请求，逐字保留，不缩写不概括 |
| `created_by` | 中书省 | 填写 `zhongshusheng` |

### context 字段

| 字段 | 填充者 | 说明 |
|------|--------|------|
| `complexity` | 尚书省 | S1 评估的五维打分结果（可选，未做 S1 可省略） |
| `domains` | 尚书省 | 任务涉及的领域列表 |
| `known_context` | 尚书省 | 用户已提供的背景信息、文件路径、错误信息、历史记录等 |

### plan 字段（中书省填充）

| 字段 | 必填 | 说明 |
|------|------|------|
| `goal` | 是 | 一句话描述任务目标，10-20 字 |
| `revision` | 是 | 当前规划版本号，从 1 开始，每次封驳+1 |
| `subtasks[]` | 是 | 子任务列表，2-6 个 |
| `subtasks[].id` | 是 | 格式 `sub-01`、`sub-02`，两位数字 |
| `subtasks[].summary` | 是 | 简短标题，10 字以内，一眼看出做什么 |
| `subtasks[].department` | 推荐 | 归属部门：兵部/户部/礼部/刑部/吏部/工部。不填则由尚书省自行执行 |
| `subtasks[].prompt` | 是 | 给执行者的完整指令，自包含、不含歧义 |
| `subtasks[].tools_hint` | 否 | 建议使用的工具列表，不确定时不填 |
| `subtasks[].depends_on` | 是 | 前置依赖的子任务 id 列表，无依赖填 `[]` |
| `subtasks[].expected_output` | 是 | 执行完成后应产出的具体成果 |

### risks 字段（中书省填充）

可选但推荐填写。列出中书省在规划时已预见到的风险和不确定性。

### review 字段（门下省填充）

| 字段 | 必填 | 说明 |
|------|------|------|
| `result` | 是 | `approved` 或 `rejected` |
| `round` | 是 | 当前审核轮次，从 1 开始 |
| `checklist.*` | 是 | 七项检查的逐项结果，true=通过，false=不通过 |
| `rejection_reason` | 打回时必填 | 具体打回理由，说明哪条检查未通过、为什么 |
| `suggestion` | 打回时必填 | 改进建议，说明希望中书省怎么改 |
| `notes` | 否 | 补充意见，审核通过时也可填写 |

### results 字段（尚书省填充）

| 字段 | 必填 | 说明 |
|------|------|------|
| `status` | 是 | `done`(全部完成)、`partial`(部分完成)、`blocked`(阻塞) |
| `subtasks[].output.status` | 是 | `done`、`blocked`、`skipped` |
| `subtasks[].output.summary` | 是 | 一句话总结做了什么 |
| `subtasks[].output.artifacts` | 否 | 产出文件清单，无产出时填 `[]` |
| `subtasks[].output.notes` | 否 | 执行中发现的注意事项 |
| `summary` | 是 | 尚书省整理的完整回奏内容，面向用户 |

---

## 使用原则

1. **只追加，不修改**。每个角色只创建自己负责的区块，不修改其他区块的内容。封驳时的 revision 是新追加一条并修改 plan 内容，不在旧版本上原地改。
2. **结构保真**。subtask 执行时 prompt 字段必须 1:1 传递给执行者（尚书省自己），不缩写不改写。
3. **审计可回溯**。封驳循环中的每一轮 revision 和 review.round 记录了完整的决策链条，不应删除。
4. **不要编造字段**。不在模板之外的字段不要自行添加。如果觉得某个字段有用但模板没有，反馈给尚书省在下一版扩展。
