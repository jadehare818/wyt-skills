---
name: plan-before-code
description: Use this BEFORE modifying any source code (new feature, bugfix, refactor, behavior change, anything that writes to source files). Forces writing a story.md (user intent) + architect-plan.md (concrete design) under .claude/plans/<slug>/, then PAUSES for user approval ("ok" or "go") before implementation begins. Provides visibility into what will change so user can intervene early. Skip only for the listed exemptions (typo fixes, pure reads, comment edits, dictated docs/config, command-only operations).
---

# plan-before-code

Lightweight code-modification protocol that produces two short artifacts under `.claude/plans/<slug>/` and pauses for user approval before any source-file write happens. Goal: user sees what you understand and what you're about to change, can correct before code is written.

## When this skill applies

ANY user request that will modify source code:
- New feature
- Behavior change
- Refactor
- Bugfix
- Anything that writes to source files

## Exemptions (skip the skill, just do the work)

1. Single-line / few-line typos and obvious one-spot fixes
2. Pure read / explore tasks (grep, read files, explain code, answer questions)
3. Comment-only changes
4. Doc / config changes the user explicitly dictated line-by-line
5. Running commands or queries that don't modify source

## Workflow

### Step 1 — Understand & propose (write two files, do not write any source code)

Pick a kebab-case slug from the request topic (e.g. `add-feedback-tag-field`, `fix-op-false-dump`, `refactor-pe-resolver`). Do NOT ask the user for the slug — just pick one.

Create directory: `.claude/plans/<slug>/`

Write **two** files:

#### `.claude/plans/<slug>/story.md` — what user wants

Capture user intent and acceptance criteria. Stay product-level, no implementation specifics.

```markdown
# Story — <slug>

## 需求理解
<one paragraph restating what the user wants, in your own words>

## 验收标准 (AC)
- [criterion 1]
- [criterion 2]
- ...

## 我注意到的关键边界 / 已知约束
- [boundary or constraint observed from conversation or quick code scan]
- ...

## 我没想清楚 / 想和你确认的
- [any open question that affects scope or behavior]
- (write "无" if nothing)
```

#### `.claude/plans/<slug>/architect-plan.md` — how you'll do it

Concrete design. Reference real files with `file:line` where relevant. No code blocks of full implementations — just the plan.

```markdown
# Architect Plan — <slug>

## 总体思路
<1-2 sentences describing the approach>

## 涉及文件
| 文件 | 现状 | 计划改动 |
|---|---|---|
| `path/to/file.ext:LINE` | <what's there now, briefly> | <what you'll change/add> |
| ... | ... | ... |

## 实施步骤
1. <specific action — file path + what changes>
2. <specific action>
3. ...

## 测试计划
- 单元测试: <which functions/components>
- 集成 / 手动验证: <how to verify>
- Edge cases to cover: <list from story.md edge cases>

## 风险 / 偏离常规模式 / 注意事项
- <any risk, deviation from project conventions, performance concern>
- (write "无" if nothing)
```

**Quality requirements (avoid lazy plans):**
- Architect plan must reference real files with paths (no placeholders like `<file>`)
- Implementation steps must be actionable without further clarification
- If you find you can't fill `涉及文件` table because you haven't read enough code, **read more code first** — do not write a vague plan

### Step 2 — Pause for review

After both files exist, print:

```
📋 已写入 .claude/plans/<slug>/
   - story.md          (需求理解 + AC)
   - architect-plan.md (设计 + 实施步骤)

请过一遍, 确认 OK 后说 "ok" 或 "go", 我开始实现。
有要改的地方直接说, 我修订后再请你确认。
```

**STOP. Do not write any source-code edits until the user explicitly says "ok" / "go" / equivalent approval.**

If the user requests changes, update the relevant file(s) and re-issue Step 2's prompt. Do not auto-proceed.

### Step 3 — Implement after approval

After explicit approval:
1. Implement the steps in `architect-plan.md` in order
2. Use the exact file paths and changes described
3. Write tests per the test plan section
4. If during implementation you discover the plan is wrong (file doesn't exist as described, approach won't work, etc.) **STOP**, update `architect-plan.md`, re-issue Step 2, get re-approval — do NOT silently deviate

When done, print which files changed and a brief summary. Suggest invoking `superpowers:verification-before-completion` or running tests before declaring success.

## Notes for the assistant

- This skill **complements** `superpowers:brainstorming`, `superpowers:writing-plans`, `superpowers:test-driven-development`, `superpowers:verification-before-completion`. You may use any of those skills to organize your thinking inside Step 1 or Step 3 — but the file-writing and pause requirements of this skill are mandatory.
- The user wants to participate in the *direction* (Step 2 review) but otherwise gives you autonomy. Don't ask micro-questions during Step 1 / Step 3 — just produce good artifacts and stay efficient.
- `.claude/plans/<slug>/` files are kept after implementation for retrospective. Don't delete them.
