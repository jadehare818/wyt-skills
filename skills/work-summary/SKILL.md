---
name: work-summary
description: This skill should be used when the user asks to "总结我的工作", "复盘一下这段时间的需求", "summarize my work", "梳理一下我做过什么", or wants to review/recap their development output over a time period. Produces a structured self-review document by analyzing git history, tracing code changes, and conducting interactive discussion with the user.
---

# work-summary

Produce a structured self-review document covering the user's development work over a specified time period. The process combines automated git/code analysis with interactive discussion to capture context that code alone cannot provide (who found the bug, what was mentor guidance, iteration history, honest self-assessment).

## When to use

- User wants to summarize work over a time range (week, sprint, month, quarter)
- User wants to prepare for a performance review, 转正答辩, or interview prep
- User says "复盘", "总结", "梳理需求", "summarize my work", "what did I do"

## Workflow overview

```
Phase 1: Discovery (git analysis)
    → identify branches, commits, time range
    → cluster into distinct requirements/features

Phase 2: Framework alignment (interactive)
    → confirm audience and depth
    → agree on document structure

Phase 3: Per-requirement deep dive (iterative)
    → code tracing + user interview per requirement
    → write up each requirement section

Phase 4: Synthesis
    → cross-cutting lessons, growth narrative
    → finalize document
```

## Phase 1 — Discovery

### Identify work scope

Determine the time range from user input. Run git analysis to discover relevant branches and commits:

```bash
# Find branches by the user's git author within the time range
git log --all --author="<git_user>" --since="<start_date>" --until="<end_date>" \
    --format="%D" | tr ',' '\n' | grep -oP '[\w/-]+' | sort -u

# For each branch, summarize commits
git log <branch> --author="<git_user>" --oneline --since="<start_date>"
```

### Cluster into requirements

Group related branches/commits into distinct requirements. Each requirement typically maps to one feature/bugfix/refactor. Present a numbered list to the user for confirmation:

```
发现了以下需求（按时间排序）：
1. [时间] 需求名 — 涉及分支: branch1, branch2
2. [时间] 需求名 — 涉及分支: branch3
...

这个列表准确吗？有遗漏或需要合并的吗？
```

## Phase 2 — Framework alignment

### Confirm audience

Ask the user: this document is for whom? Common audiences:
- **Self** (复盘): maximum honesty, include mistakes and lessons
- **转正/绩效**: emphasize impact and growth, attribute correctly
- **面试**: emphasize technical depth and problem-solving

### Agree on document structure

Default structure (adaptable per audience):

```markdown
# 工作总结 — [时间段]

## 一、我的位置
- System architecture context (where does my work sit)
- Key services and their relationships
- My scope and role

## 二、业务理解
- Product capability layer (what models/features exist)
- User value layer (why each requirement matters to users)
- Data awareness layer (metrics, impact numbers)

## 三、需求详解
### 需求 N: [名称]
- 背景 (what problem existed)
- 方案 (technical approach + data flow)
- 迭代过程 (how many rounds, what went wrong)
- Lessons
- 诚实标注 (who did what — mentor vs self vs AI-assisted)

## 四、踩坑与成长
## 五、还不够的地方
```

## Phase 3 — Per-requirement deep dive

For each requirement, follow this sub-workflow:

### 3.1 Code tracing

Read the branch diff to understand what changed:

```bash
git log <branch> --oneline
git diff <base>..<branch> --stat
```

For key files, read the actual implementation to understand the design:
- Identify the data flow (entry point → processing → output)
- Identify cross-service interactions
- Note configuration patterns (nacos, asynccache, feature flags)

### 3.2 User interview (interactive)

Code analysis reveals WHAT was done, but not the full story. Ask the user about:

1. **Iteration history**: "这个需求迭代了几轮？每轮遇到什么问题？" — static code only shows the final state
2. **Attribution**: "这个方案是你自己设计的还是 mentor 指导的？哪些 bug 是你自己定位的？" — for honest annotation
3. **Context code can't provide**: deployment issues, upstream/downstream communication, requirement changes mid-flight
4. **Debugging process**: "发现问题后你是怎么排查的？" — the diagnostic process is often more valuable than the fix

Ask at most 2-3 questions per requirement per round. Don't overwhelm.

### 3.3 Write up

For each requirement, produce a section covering:

| Section | Content | Source |
|---------|---------|--------|
| 背景 | What problem existed, why it matters | Code + user context |
| 方案 | Technical design, data flow diagram (ASCII), key decisions | Code tracing |
| 迭代过程 | Rounds of deployment/testing/fixing (if applicable) | User interview |
| 深入理解 | Technical insights worth explaining (patterns, tradeoffs) | Code + synthesis |
| Lessons | What was learned — applicable to future work | Discussion |
| 诚实标注 | Who did what (mentor/self/AI) | User interview |

**Writing principles:**
- Data flow diagrams in ASCII (using `→`, `│`, `├──`, `▼`) — show the path a request/parameter takes
- Explain WHY, not just WHAT — "用户可以控制视频画质" not "加了个参数"
- Technical depth should match audience: self-review gets full detail, interview version gets polished highlights
- Lessons should be generalizable, not just "I fixed this specific bug"

## Phase 4 — Synthesis

After all requirements are written:

### Cross-cutting themes

Look across requirements for patterns:
- Recurring mistake types (e.g., "改了一个 worker 忘了其他 worker")
- Skills that improved over the period
- Technical patterns mastered (nacos asynccache, ctx usage, distributed parameter passing)

### Growth narrative

Frame growth as: **之前的思维 → 现在的思维**

Example:
| Before | After |
|--------|-------|
| 改了一个地方就觉得完成了 | 先查清楚有多少条并行路径，全部改完才算完 |
| 本地单测过了就提 MR | 必须在实验环境端到端跑通才算验证 |

### Gaps and next steps

Honest self-assessment of what's still lacking — not self-deprecation, but clear-eyed identification of growth areas.

## Output

Write the final document to a path the user specifies, or default to:
`~/Documents/self-review-<YYYYMM>.md`

## Tips for effective execution

1. **Don't rush Phase 3.2** — the user interview is where the document's value comes from. Code can be read anytime; the user's memory of debugging sessions fades.
2. **One requirement at a time** — fully discuss and write up requirement N before moving to N+1.
3. **Verify code claims** — if you're about to describe a function or data flow, read the actual code first. Don't hallucinate based on branch names.
4. **Relative dates → absolute dates** — when the user says "上周" or "周四", convert to YYYY-MM-DD in the document.
5. **ASCII data flow diagrams** — these are the most valuable part of deep technical writeups. Show how a parameter flows from API entry through persistence/queuing to final consumption.

## Additional resources

### Reference files

- **`references/deep-dive-patterns.md`** — Patterns for explaining technical decisions (ctx vs struct, lock analysis, gate+builder, two-phase validation)
