---
name: iterative-discussion
description: "任务执行前的迭代讨论与确认流程。任何任务开始前必须调用。通过批量提问、单独追问、方案确认、spec 确认、执行后追问五阶段，确保对齐后再动手。"
---

# 迭代讨论与确认流程

将 brainstorming 的"提问-确认"模式泛化到所有通用任务。任何任务开始前必须走完本流程，未获用户确认不得动手实施。

<HARD-GATE>
Do NOT invoke any implementation skill, write any code, modify any file, or take any execution action until you have completed the clarifying + proposal + confirmation stages and the user has approved. This applies to EVERY task regardless of perceived simplicity.
</HARD-GATE>

## Anti-Pattern: "This Is Too Simple To Need Discussion"

Every task goes through this process. Reading a file, fixing a variable name, a one-line patch — all of them. "Simple" tasks are where unexamined assumptions cause the most wasted work. The discussion can be short (a few questions for truly simple tasks), but you MUST ask and get approval before acting.

## Checklist

You MUST create a task for each of these items and complete them in order:

1. **批量提问** — 收到任务后立即用 `AskUserQuestion` 一次提 2-4 个关键问题
2. **单独追问** — 对仍模糊的点逐个单独追问（一次一问），直到无歧义；若阶段①已充分可跳过
3. **方案+确认** — 给方案（数量按任务复杂度 AI 自判）+ `AskUserQuestion` 确认
4. **spec+确认** — 按任务类型生成 spec 文档（可省见下表）+ `AskUserQuestion` 确认
5. **执行+追问** — 执行任务 → 自检 → 报告 → `AskUserQuestion` 问"是否有下一阶段任务/补充/调整"

## 五阶段流程

```
① 批量提问 → ② 单独追问 → ③ 方案+确认 → ④ spec+确认 → ⑤ 执行+追问下一阶段
```

**阶段① 批量提问**
收到任务后立即用 `AskUserQuestion` 单次调用，提 2-4 个关键问题，覆盖：目的、约束、成功标准、歧义点。每问 2-4 个选项，必有"其他"。

**阶段② 单独追问**
审阅用户回答，对仍模糊的点逐个单独追问（每次 `AskUserQuestion` 1 问），直到无歧义。若阶段①回答已充分，本阶段可跳过。

**阶段③ 方案+确认**
按任务复杂度给方案：明显只有一种做法就给 1 个方案 + 权衡；有多条路径就给 2-3 个对比。用 `AskUserQuestion` 让用户选/改/否。未确认不动手。

**阶段④ spec+确认**
按下方"任务类型规范表"生成 spec 文档。用 `AskUserQuestion` 让用户确认 spec；用户要改就改完再问一次。**禁止用 NotifyUser 做确认**（它会中断对话）；NotifyUser 仅用于通知 spec 已写好供审阅，确认动作仍走 `AskUserQuestion`。

**阶段⑤ 执行+追问**
执行任务 → 自检 → 报告结果 → 用 `AskUserQuestion` 问"是否有下一阶段任务要求/补充/调整/重做/结束"。用户说"结束"才结束；说有则回到阶段①。

## 任务类型规范表

按任务类型决定 spec 文档详略、提问侧重、验证标准：

| 任务类型 | spec 文档要求 | 提问侧重 | 验证标准 |
|---|---|---|---|
| **debug 任务** | 可省 spec；若根因复杂则写 spec.md（症状+根因+修复方案） | 复现条件、影响范围、是否需保留临时绕过 | 原症状消失、无回归 |
| **工程重构** | 必出 spec.md + checklist.md | 重构动机、对外接口是否变、回滚策略 | 行为不变 + 测试全绿 |
| **功能实现** | 必出三件套（spec.md + checklist.md + tasks.md） | 目的、边界、成功标准、歧义点 | 满足 spec 验收项 |
| **运维任务** | spec.md 几行（操作+影响范围）；高危操作必出 checklist.md | 影响范围、回滚、时间窗 | 操作完成 + 系统正常 |
| **其他/不确定** | AI 判断；倾向从简 | 目的、约束 | 目标达成 |

**spec 文档可省的明确条件**：
- debug 任务根因简单
- 任务本身是"制作技能/读文件/单行修复"等极简任务
- 用户明确说"跳过 spec 直接做"

满足以上任一条件时，跳过阶段④的文档生成，但阶段③方案确认和阶段⑤追问不省，整体走 ①②③⑤ 四阶段。

spec 文档存放：`docs/superpowers/specs/YYYY-MM-DD-<topic>.md`（用户偏好可覆盖路径）。

## AskUserQuestion 使用规则

| 阶段 | 单次调用问题数 | 每问选项数 |
|---|---|---|
| ① 批量提问 | 2-4 | 2-4 + 必有"其他" |
| ② 单独追问 | 1 | 2-4 |
| ③ 方案确认 | 1-2（选方案 + 是否需调整） | 2-4 |
| ④ spec 确认 | 1 | 2-4 |
| ⑤ 执行后追问 | 1 | 选项：补充/调整/重做/结束 |

**禁止**：用 NotifyUser 做确认；用纯文本提问代替 `AskUserQuestion`（除非问题本质上是开放讨论而非选项选择）。

## 与 brainstorming 技能的关系

- 不修改 brainstorming 技能（它是创意/新功能设计专用，保留）
- 本技能是通用任务流程，**不绑定 writing-plans**（不像 brainstorming 终态必须是 writing-plans）。任务执行完即结束，或回到阶段①
- 若任务确实是"创意/新功能设计"，AI 可建议改用 brainstorming

## 关键原则

- **批量优先** — 阶段①必须批量提问，不逐个问
- **AI 自判方案数** — 不凑数，按复杂度给 1-3 个
- **弹性 spec** — 按任务类型定文档详略，不为极简任务强加三件套
- **YAGNI** — 移除不必要的特性
- **不中断对话** — 确认一律 `AskUserQuestion`，不用 NotifyUser
- **结束后必追问** — 阶段⑤是硬要求，不得自行结束对话

## 验证方式（技能上线后自测）

用 3 类任务自测：
1. **极简任务**（如"读 memory.md"）— 验证流程走得动但不啰嗦，spec 可省
2. **中等任务**（如"给某模块加日志"）— 验证 spec.md+checklist.md 产出
3. **模糊任务**（如"优化一下性能"）— 验证阶段②追问能逼出细节

每类跑一遍，检查：
- 是否每阶段都用了 `AskUserQuestion`
- 是否没漏触发（user_rule 强制）
- 是否没用 NotifyUser 做确认
- 阶段⑤是否追问了"下一阶段"
