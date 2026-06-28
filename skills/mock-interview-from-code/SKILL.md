---
name: mock-interview-from-code
description: Use when the user wants to rehearse a technical interview based on a real codebase they've worked on (实习/工作项目面试准备). Produces multi-turn Q&A dialogues grounded in code facts (not memory), with side-commentary explaining what each question tests and what traps the interviewer might set. Triggers include 「模拟面试」「面试准备」「复盘项目准备面试」「我拿这段实习去面试会被问什么」「mock interview」「interview prep from project」. Do NOT use for: generic interview questions, JD/resume writing, behavioral-only interviews, or topics the user has not actually worked on.
---

# Mock Interview From Code

A discipline for producing high-fidelity mock interview Q&A material from a real codebase the user has worked on.

The user wants:
- Predictable interviewer questions for that project
- Polished answers in their own voice
- Side-commentary explaining what each question tests
- Coverage broad enough that someone unfamiliar with the project could understand it after reading the material

## Core principle

**代码事实 > 记忆总结 > 推理猜测.**

Three priority tiers, strictly descending. Never write Q&A based on memory or existing summary docs alone — always dispatch an Explore subagent to verify against current code first. Memory files (e.g. `MEMORY.md`, `arkive`, design docs) are starting points for *what to verify*, not authoritative sources for *what to write*.

## The 5-phase workflow

### Phase 0 · Calibrate before writing (AskUserQuestion, 4 dimensions)

**Never start writing until all four dimensions are answered.** Use AskUserQuestion with a single call containing all four questions. Each question gets 3–4 options. Mark one option `（推荐）` per question.

| Dimension | Why it matters | Example options |
|-----------|----------------|-----------------|
| **面试场景 (scenario)** | Determines question depth and jargon density | 大厂校招 / 公司内部转正 / 独角兽社招 / 创业公司 |
| **回答定位 (answer positioning)** | Determines whether answers should reveal weak spots | 理想答案版 / 真实水平版 / 混合版（推荐） |
| **文档结构 (structure)** | Determines teaching value | 纯对话 / 对话+旁白点评（推荐） / 对话+背景知识卡 |
| **生成节奏 (cadence)** | Determines correction windows | 一个主题一停（推荐） / 几题一停 / 一次写完 |

Each calibration question changes downstream output significantly. Skipping any one risks a full rewrite.

After answers come in, plan the topic list (typically 3–6 themes) and confirm with the user before starting Phase 1 on the first theme.

### Phase 1 · Dispatch Explore for facts (every theme, every time)

Before writing any Q&A for a theme, dispatch an `Explore` (or read-only general-purpose) subagent to gather code facts. The Explore prompt must follow this structure:

```
仓库在 <absolute path>. 我需要写一份面试问答材料，主题是 <theme>.
请按以下子主题分节调查，每节给我:
- 相关文件路径（带行号锚点）
- 关键代码片段（短摘录）
- 字段/枚举/状态的精确含义
- 不确定的明确写"未找到"或"推测：…"

**主题 A：<具体调查任务>**
- 搜 <具体关键词1>、<具体关键词2>
- 验证假设：<列出你想验证或推翻的具体假设>
- 必要的 git 命令: <如 git show <sha>>

**主题 B：<...>**
...

输出要求:
- markdown 按主题分节
- 每个事实带 file_path:line_number
- 不要建议、不要点评，只要事实
- 看不到就说看不到，不要编
```

Critical constraints on the Explore prompt:
1. **List specific keywords and file hints** — don't say "find watermark logic", say "search `watermark`, `blind_watermark`, `dark_watermark`, `steganography`".
2. **State hypotheses to verify or refute** — pulled from the user's memory files, prior summaries, or your own assumptions. Forces the subagent to disprove rather than rubber-stamp.
3. **Forbid suggestions/judgments** — "不要建议、不要点评，只要事实". LLMs default to "helpfulness mode" and will pollute the fact ledger with their analysis unless explicitly told not to.
4. **Allow "未找到"** — better to surface gaps than fabricate. Truthfulness > completeness in fact gathering.

If the first Explore dispatch leaves gaps that surface during writing, **dispatch a second round** focused on the gaps. Don't try to write around missing facts.

### Phase 2 · Write the theme (three-block structure per Q&A turn)

Each Q&A round has this exact shape:

```markdown
**面试官**：[Question, often a probe — "what is X?", "why X?", "how does X handle Y?"]

**你**：[Answer in plain language. Code paths, function names, struct field names parenthesized or in code blocks. Each factual claim should be traceable to the Explore fact ledger.]

> 🎯 **旁白·<one-line characterization of what this question tests>**
> [3 paragraphs:]
> 1. 这道题在考什么 — the latent dimension the interviewer is probing (system design instinct, debugging discipline, business judgment, etc.)
> 2. 答案的关键点 — the 2-3 things the answer absolutely must hit
> 3. 可能的追问陷阱 — predicted follow-up questions and short answers to each
```

Interleave these two elements at logical moments:
- **📖 背景知识卡** — insert when a domain term first appears or a system topology is first introduced. Can include ASCII diagrams, term comparison tables, memory mnemonics. Aimed at "the reader who has never done this project".
- **末尾考点地图** — end-of-chapter summary table listing all sub-questions, what they test, and must-hit keywords. Aimed at "the user reviewing before the actual interview".

### Phase 3 · Language style for the main answer

The main answer (the **你**: line) must be readable by someone who **doesn't know the codebase**. Code details get parenthesized.

| ✅ Do | ❌ Don't |
|------|---------|
| "队列底层用了三个 Redis 数据结构组合" | "我们用 ZSET+SET+ZSET 实现 dirty queue" |
| "并发自适应调整，下游一抖就退" | "AIMD targetConcurrency *= RetreatFactor" |
| "...（实现在 `mw_minmax_content_gen_task/controller.go`）" | "代码在 `app/ark-async-gateway/pkg/worker/mw_minmax_content_gen_task/controller.go:181`" |
| "队列用 ZSET 存任务、按时间戳排序" + 后续括号给文件路径 | 一上来就甩 `pkg/redisqueue/script.go:42` |

The rule: **plain language carries the narrative; code details ride along in parentheses or code blocks**. Both audiences served simultaneously — the non-coder interviewer can follow, the coder interviewer can verify.

### Phase 4 · Treat gaps in the code as answering material

When Explore reveals that the codebase **does not have feature X** (no idempotency key, no circuit breaker, no webhook signature, no retry backoff, no test coverage…), do **not** hide it. Build a Q&A round around it. Use this three-part structure:

```
**你**：实话说，<未做 X>. 我查过代码：<state the evidence that confirms it's not there, file:line if possible>.

为什么没做：<one or two of>
- 业务原因（成本不高/可逆/低频）
- 历史包袱（早期选型/灰度中）
- 兜底机制（mesh 熔断/上层签名/sdk 重试）

如果要做我会怎么做：<concrete 2-3 step plan>
```

This is the **highest-leverage answering pattern**:
- Signals honesty (interviewer's #1 trust check)
- Signals trade-off awareness (knows *why* it's not done)
- Signals improvement mindset (has a plan, not a complaint)

Reserve 2-3 of these per theme. Common gap territories: idempotency, retry strategy, circuit breaking, webhook signing, test coverage, observability gaps, schema migrations, multi-region failover.

### Phase 5 · One theme, then stop

After writing each theme:
1. Update its file (e.g. `01-<theme>.md`) — location/naming decided by user, do not hardcode.
2. Summarize what was added (turn count, new background cards, key insights).
3. Ask the user to verify **three things specifically**:
   - Factual claims and judgment calls in the side-commentary that depend on user's lived experience
   - Whether "gap" claims (没做 X) match user's actual recollection
   - Style/length/density preferences for the next theme
4. **Stop and wait**. Do not proceed to the next theme until the user confirms.

User feedback typically triggers:
- Style adjustments (more plain language, less code, etc.)
- A second Explore dispatch to cover new gaps the user surfaces
- Local rewrites of specific Q&A rounds where the side-commentary judgment was off

Resist the urge to write all themes in one pass. Your "what makes a good answer" judgment is always weaker than the user's judgment about their own work. Build correction windows into the cadence.

## Cross-cutting principles (apply throughout all phases)

1. **代码事实 > 记忆总结 > 推理猜测.** Three tiers, strictly descending. When the user's `MEMORY.md` or design docs say "feature X exists with Y states", treat that as a **hypothesis to verify**, not a fact to cite.

2. **Serve two audiences simultaneously.** Main answer = plain language (the non-coder interviewer); parentheses/code blocks = code details (the coder interviewer who wants to verify).

3. **The side-commentary (旁白点评) is the highest-value part of the document.** Without it, you've just written a dialogue script. With it, you've written teaching material. Every side-commentary block must answer "this question is testing X".

4. **Predict follow-ups, don't wait to be hit.** Side-commentary should always include "可能的追问陷阱" with short answers. The user reading the doc once should leave with contingency plans, not just a single answer.

5. **End each theme with a closed-loop diagram.** The final Q&A in each theme should be "walk me through the full lifecycle of <X>" or "draw <X> on a whiteboard". This is the moment where everything else gets tied together and the interviewer signs off mentally.

6. **Side-commentary judgments must be verifiable by the user.** When you write "这是典型的安全/可用性 trade-off, 候选人应该讲出 fail-close 还是 fail-open"—make it concrete enough that the user can say "no, in our case it's actually fail-open because XYZ" and you can correct it.

## Anti-patterns (don't do this)

| Anti-pattern | Why it hurts |
|--------------|--------------|
| Writing Q&A from `MEMORY.md` directly without Explore | High risk of citing facts that were true 6 months ago, not today |
| Listing code paths in the main answer body | Non-coder interviewer can't follow; reads as "I memorized paths" not "I understand" |
| Padding the answer to feel comprehensive | Reads as nervous over-explaining; weak interviewees do this |
| Generic side-commentary like "这道题考察基础" | Useless. Side-commentary must name the specific latent dimension. |
| All-or-nothing answer positioning | Real users want a mix — strong on what they know, honest about gaps. |
| Writing all themes in one pass without checkpoints | You will write the second theme on a wrong understanding of the user's voice from the first. |
| Treating "Explore found nothing" as a problem to hide | Use the gap as a Q&A round (see Phase 4). |
| Skipping the closed-loop end-of-theme diagram | The user loses the "I can complete the picture" rehearsal moment. |

## Output naming and location

**Do not hardcode this.** Confirm with the user before Phase 1. Common patterns:

- `interview-prep/01-<theme-slug>.md`, `02-<theme-slug>.md`, ... — recommended for multi-theme projects
- Single combined `interview.md` — fine for compact preparations
- User-specified location

Default suggestion if user doesn't specify: `interview-prep/NN-<theme-slug>.md` under the project root, where NN is two-digit ordering, theme-slug is kebab-case English.

## Theme catalog inspiration

For a typical 实习/工作项目, 5–6 themes usually suffice. Common axes:

- **工程化主线** — system design end-to-end (queues, workers, rate limits, observability) — usually the longest theme
- **业务理解** — domain concepts, what users actually do with the product, why decisions were made
- **新功能上线复盘** — pick one concrete feature commit, walk through all four layers (whitelist, params, encoding, rate limit, etc.)
- **稳定性与排障** — how you observe, debug, recover; war stories
- **协作与流程** — version control, code review, release, on-call, cross-team
- **架构治理** — refactoring, config-as-code, migration, gradual rollout

These are starting points, not a fixed template. Theme selection should come from the user's actual work, not a checklist.

## A note on this skill's heritage

This skill was distilled from a real session preparing 字节大模型方舟 (Volcengine Ark MaaS) AIGC 视频生成 interview material. The first theme (AIGC 视频生成的工程化) ran two Explore dispatches, 23 Q&A rounds, 14 side-commentary blocks, and 2 background knowledge cards across two iterations. The principles above are exactly the ones that survived the iterations. If a future user reports the workflow felt mechanical or the output felt thin, that is the signal to deepen Phase 1's Explore prompts and add more "gap as material" rounds in Phase 4 — not to shorten the workflow.
