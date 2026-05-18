# 三省六部制 AI Agent 架构

> 基于唐朝三省六部制设计的 AI Agent 架构，实现「规划→审核→执行→回奏」的完整工作流程。
> 专为 [Hanako](https://github.com/lilimozi/openhanako) AI 助手项目设计。

---

## 概述

传统 AI Agent 面对复杂任务的问题：直接执行，缺乏规划和审核，导致失败率高、质量不稳定。

三省六部制将 Agent 工作流拆解为三个独立角色：

| 角色 | 映射 | Agent | 职责 |
|------|------|-------|------|
| **皇帝** | 决策者 | 用户 | 最终拍板，决定方向 |
| **宰相** | 幕僚 | Butter | 理解意图、把控温度、译奏汇报。直接向皇帝负责 |
| **中书省** | 规划 | 指定 agent | 拆解任务，制定方案，输出 Plan Object |
| **门下省** | 审核 | 指定 agent | 审核方案，逐项检查，封驳不合格的 |
| **尚书省** | 调度 | 主 agent | 分配任务，居中调度，六部执行，汇总回奏 |

执行阶段由六个专业部门（六部）分工完成，每个部有独立的领域知识注入。

## 架构图

```
用户请求 → 尚书省（S0/S1 评估）
         → 中书省（规划）→ Plan Object
         → 门下省（审核）→ 通过/封驳（最多 3 轮）
         → 尚书省（调度执行）
              ├─ 兵部 - 技术实现（代码、调试、架构）
              ├─ 户部 - 数据分析（数据处理、统计、图表）
              ├─ 礼部 - 内容创作（文档、文案、翻译）
              ├─ 刑部 - 安全合规（安全审计、风险检查）
              ├─ 吏部 - 流程管理（任务跟踪、协调）
              └─ 工部 - 部署运维（CI/CD、环境配置）
         → 回奏
```

## 适用场景

| 适用 | 不适用 |
|------|--------|
| 多步骤复杂任务 | 简单问答 |
| 高不确定性任务 | 打招呼、查天气 |
| 多领域交叉任务 | 单步操作 |
| 涉及不可逆操作的任务 | 日常闲聊 |
| 需要外部审核确认的任务 | 已有成熟解法的已知流程 |

## 通信协议

三省之间通过结构化的 **Plan Object**（YAML）传递信息：

- 中书省填充 `plan` 区块（任务拆解、子任务清单）
- 门下省填充 `review` 区块（七项审核清单逐条检查）
- 尚书省填充 `results` 区块（执行结果汇总）

Plan Object 也是中断恢复的状态载体——在每个阶段切换时自动保存，确保流程中断后可恢复。

## 快速开始

1. 将 `SKILL.md` 安装为你的 agent 技能
2. 配置中书省（规划 agent）和门下省（审核 agent）的指向
3. 当遇到复杂任务时，按照 SKILL.md 中的三省协作流程执行

### 参考文件

| 文件 | 内容 |
|------|------|
| `SKILL.md` | 主技能文件——三省六部完整编排流程 |
| `references/plan-object-format.md` | Plan Object 完整 YAML 格式规范 |
| `references/zhongshusheng-prompt.md` | 中书省（规划 agent）subagent 指令模板 |
| `references/menxiasheng-prompt.md` | 门下省（审核 agent）subagent 指令模板 |
| `references/liubu-design-outline.md` | 六部架构设计概要 |
| `references/liubu/bingbu.md` | 兵部（代码实现）prompt 模板 |
| `references/liubu/hubu.md` | 户部（数据分析）prompt 模板 |
| `references/liubu/libu-content.md` | 礼部（内容创作）prompt 模板 |
| `references/liubu/xingbu.md` | 刑部（安全合规）prompt 模板 |
| `references/liubu/libu-admin.md` | 吏部（流程管理）prompt 模板 |
| `references/liubu/gongbu.md` | 工部（部署运维）prompt 模板 |

## 停止条件体系

三省六部流程内置多层停止条件：

- **三省层**：封驳 3 轮上限、subagent 超时/空输出降级
- **六部层**：单子任务重试 2 轮上限、subagent 超时 fallback
- **全局层**：100K token 预算、循环退化检测、用户中断、尚书省主动权

## 中断恢复

Plan Object 自动持久化到 `.sanbu/state/`，确保流程中断后可恢复。在每个阶段切换时保存，正常完成后自动清理。
