迭代讨论强制规则：
任何任务开始前，必须先调用 `iterative-discussion` 技能走完确认流程，否则不得动手实施。
AI 按任务类型（debug/工程重构/功能实现/运维/其他）走对应规范：
- debug 任务：可省 spec；根因复杂则写 spec.md（症状+根因+修复方案）
- 工程重构：必出 spec.md + checklist.md
- 功能实现：必出三件套（spec.md + checklist.md + tasks.md）
- 运维任务：spec.md 几行；高危操作必出 checklist.md
- 其他/不确定：AI 判断，倾向从简

确认一律用 AskUserQuestion，禁止用 NotifyUser 做确认（避免中断对话）。
任务完成后必须用 AskUserQuestion 追问"是否有下一阶段任务/补充/调整"，用户明确说"结束"才结束。
