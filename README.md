# Iterative Discussion

[English](#english) | [中文](README.zh-CN.md)

A TRAE skill that enforces an **iterative discussion and confirmation workflow before any task execution**. Generalizes the "ask-before-act" pattern from `brainstorming` to all task types (debug, refactoring, feature implementation, ops, etc.), ensuring alignment with the user before writing any code or modifying any file.

> **Read this in: [中文](README.zh-CN.md)**

---

<a name="english"></a>
## What It Does

Any task — no matter how small — must go through a 5-stage confirmation flow before implementation begins, and must end with a follow-up question rather than silently terminating the conversation.

The skill itself is just a prompt (a `SKILL.md` file) plus a `user_rule` that forces TRAE to invoke it on every task. There is no runtime code; the mechanism is:

1. **SKILL.md instruction text** — defines when to ask, what to ask, how to ask
2. **`AskUserQuestion` tool calls** — renders interactive question cards, returns user choices to the AI
3. **HARD-GATE** — a hard constraint that forbids any implementation action until the user approves

## 5-Stage Flow

```mermaid
flowchart LR
    A[Stage 1<br/>Batch Questions<br/>2-4 questions at once] --> B{Still ambiguous?}
    B -- yes --> C[Stage 2<br/>Follow-up One-by-One<br/>1 question per call]
    B -- no --> D[Stage 3<br/>Proposal + Confirm<br/>1-3 options, AI-judged count]
    C --> D
    D --> E[Stage 4<br/>Spec Doc + Confirm<br/>elastic by task type]
    E --> F[Stage 5<br/>Execute + Self-check + Report]
    F --> G{Next stage<br/>needed?}
    G -- yes --> A
    G -- "no (user says end)" --> H[End]
```

## Task-Type-Specific Spec Rules

Spec document detail is elastic based on task type — no one-size-fits-all three-document requirement:

| Task Type | Spec Document | Question Focus | Acceptance |
|---|---|---|---|
| **Debug** | May omit; write `spec.md` (symptom+root cause+fix) if root cause is complex | Repro conditions, blast radius, temp workaround needed? | Symptom gone, no regression |
| **Refactoring** | `spec.md` + `checklist.md` | Motivation, interface changes, rollback strategy | Behavior unchanged + tests green |
| **Feature** | Full set: `spec.md` + `checklist.md` + `tasks.md` | Purpose, boundaries, success criteria, ambiguities | Meets spec acceptance items |
| **Ops** | `spec.md` (a few lines); `checklist.md` for high-risk ops | Blast radius, rollback, time window | Op complete + system healthy |
| **Other/Unclear** | AI judgment, lean simple | Purpose, constraints | Goal achieved |

**Spec may be omitted when**: simple debug root cause, the task itself is "make a skill / read a file / one-line fix", or the user explicitly says "skip spec". Even then, Stage 3 (proposal confirm) and Stage 5 (follow-up question) are never skipped.

## Relationship with `brainstorming`

- Does **not** modify `brainstorming` — it remains the specialized skill for creative/new-feature design
- `iterative-discussion` is for **general tasks** and does **not** mandate `writing-plans` as terminal state (unlike `brainstorming`)
- If the task is genuinely "creative / new feature design", the AI may suggest switching to `brainstorming`

## Pros & Cons

**Pros**
- Eliminates wasted work from unexamined assumptions — even "simple" tasks get a sanity check
- Batch questioning (Stage 1) is faster than one-at-a-time for gathering key details
- Elastic spec rules avoid burdening trivial tasks with three-document overhead
- Mandatory follow-up question prevents the AI from silently ending the conversation
- Confirmation via `AskUserQuestion` (not `NotifyUser`) avoids dialog interruption

**Cons**
- Every task pays an upfront questioning cost — can feel heavy for truly trivial operations
- Requires a companion `user_rule` to guarantee triggering; relying on the skill description alone may miss fires
- The AI must judge task type and spec detail level — quality depends on the model

## Installation

### 1. Copy the skill

Place `SKILL.md` under your TRAE skills directory:

```
<trae-config-dir>/skills/iterative-discussion/SKILL.md
```

On Windows the config dir is typically `C:\Users\<you>\.trae-cn\` (CN build) or `C:\Users\<you>\.trae\` (international).

### 2. Add the user rule

Copy the contents of [`user-rule.md`](user-rule.md) into a new file under:

```
<trae-config-dir>/user_rules/rule-<timestamp>.md
```

This forces TRAE to invoke `iterative-discussion` on every task. Without it, the skill relies on the AI's self-judgment and may not fire.

### 3. Verify

Start a new TRAE session and give it any task. The AI should immediately call `AskUserQuestion` with 2-4 questions before doing anything else.

## Files

- [`SKILL.md`](SKILL.md) — the skill definition (5-stage flow, HARD-GATE, task-type table, AskUserQuestion rules)
- [`user-rule.md`](user-rule.md) — the mandatory trigger rule
- [`README.md`](README.md) — this file (English)
- [`README.zh-CN.md`](README.zh-CN.md) — Chinese version

## License

No license file is included. Default copyright applies to the author. Feel free to copy and adapt for personal use.
