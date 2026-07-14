# 迭代讨论（Iterative Discussion）

[中文](#中文) | [English](README.en.md)

一个 TRAE 技能，**在任何任务执行前强制走迭代讨论与确认流程**。把 `brainstorming` 的"先问后做"模式泛化到所有任务类型（debug、重构、功能实现、运维等），确保动手前与用户对齐。

> **切换语言：[English](README.en.md)**

---

<a name="中文"></a>
## 它做什么

任何任务——无论多小——在动手实施前都必须走完 5 阶段确认流程，且结束后必须追问用户是否还有下一阶段，而不是自行结束对话。

技能本身只是一段 prompt（一个 `SKILL.md` 文件）加一条 `user_rule` 强制 TRAE 每个任务都调用它。没有任何运行时代码，机制是：

1. **SKILL.md 指令文本** —— 规定何时问、问什么、怎么问
2. **`AskUserQuestion` 工具调用** —— 渲染交互式问题卡片，把用户选择回传给 AI
3. **HARD-GATE 强制门** —— 硬约束，用户未批准前禁止任何实施动作

## 五阶段流程

```mermaid
flowchart LR
    A[阶段1<br/>批量提问<br/>一次2-4个问题] --> B{仍有歧义?}
    B -- 是 --> C[阶段2<br/>单独追问<br/>每次1问]
    B -- 否 --> D[阶段3<br/>方案+确认<br/>1-3个方案, AI自判数量]
    C --> D
    D --> E[阶段4<br/>spec文档+确认<br/>按任务类型弹性]
    E --> F[阶段5<br/>执行+自检+报告]
    F --> G{有下一阶段?}
    G -- 是 --> A
    G -- "否（用户说结束）" --> H[结束]
```

## 按任务类型的 spec 规范

spec 文档详略按任务类型弹性调整——不搞一刀切的三件套：

| 任务类型 | spec 文档 | 提问侧重 | 验证标准 |
|---|---|---|---|
| **debug 任务** | 可省；根因复杂则写 `spec.md`（症状+根因+修复方案） | 复现条件、影响范围、是否需保留临时绕过 | 原症状消失、无回归 |
| **工程重构** | 必出 `spec.md` + `checklist.md` | 重构动机、对外接口是否变、回滚策略 | 行为不变 + 测试全绿 |
| **功能实现** | 必出三件套：`spec.md` + `checklist.md` + `tasks.md` | 目的、边界、成功标准、歧义点 | 满足 spec 验收项 |
| **运维任务** | `spec.md` 几行；高危操作必出 `checklist.md` | 影响范围、回滚、时间窗 | 操作完成 + 系统正常 |
| **其他/不确定** | AI 判断，倾向从简 | 目的、约束 | 目标达成 |

**spec 可省的明确条件**：debug 根因简单、任务本身是"制作技能/读文件/单行修复"等极简任务、或用户明确说"跳过 spec 直接做"。即便省 spec，阶段 3（方案确认）和阶段 5（追问）也不省。

## 与 `brainstorming` 的关系

- **不修改** `brainstorming` —— 它仍是创意/新功能设计的专用技能
- `iterative-discussion` 面向**通用任务**，**不强制**以 `writing-plans` 为终态（与 brainstorming 不同）
- 若任务确实是"创意/新功能设计"，AI 可建议改用 `brainstorming`

## 优缺点

**优点**
- 消除未审视假设导致的返工——哪怕"简单"任务也过一道 sanity check
- 阶段 1 批量提问（一次 2-4 问）比逐个问更快收集关键细节
- 弹性 spec 规则避免极简任务被三件套文档压垮
- 阶段 5 强制追问，防止 AI 自行结束对话
- 确认一律用 `AskUserQuestion`（不用 `NotifyUser`），避免中断对话

**缺点**
- 每个任务都有前置提问成本——对真正琐碎的操作可能显重
- 需配套 `user_rule` 才能保证触发；仅靠技能 description 可能漏触发
- AI 需判断任务类型和 spec 详略——质量依赖模型能力

## 安装

### 1. 复制技能文件

把 `SKILL.md` 放到 TRAE 技能目录下：

```
<trae配置目录>/skills/iterative-discussion/SKILL.md
```

Windows 上配置目录通常是 `C:\Users\<你>\.trae-cn\`（国内版）或 `C:\Users\<你>\.trae\`（国际版）。

### 2. 添加 user_rule

把 [`user-rule.md`](user-rule.md) 的内容复制到：

```
<trae配置目录>/user_rules/rule-<时间戳>.md
```

这条规则强制 TRAE 每个任务都调用 `iterative-discussion`。没有它，技能只能靠 AI 自主判断触发，可能漏触发。

### 3. 验证

开一个新的 TRAE 会话，给任意任务。AI 应当立即调用 `AskUserQuestion` 提 2-4 个问题，再做其他事。

## 文件

- [`SKILL.md`](SKILL.md) —— 技能定义（5 阶段流程、HARD-GATE、任务类型表、AskUserQuestion 规则）
- [`user-rule.md`](user-rule.md) —— 强制触发规则
- [`README.md`](README.md) —— 本文件（中文，GitHub 首页默认显示）
- [`README.en.md`](README.en.md) —— 英文版

## 许可证

未包含 LICENSE 文件，默认归作者所有。欢迎复制和改编供个人使用。
