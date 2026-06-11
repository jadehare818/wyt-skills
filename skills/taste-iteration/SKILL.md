---
name: taste-iteration
description: Use this skill whenever the user wants to co-create something taste-driven by iterating from a reference example or vague subjective feedback — including literary writing, sentence polishing, naming (products/variables/files), copywriting, brand voice, slogans, design tone, UI text, or even code style choices. Trigger when the user says things like "类似的/相似的/再来几个/换一个/换个味/多来点", or critiques output with subjective adjectives ("装/腻/平/俗/不够文艺/不对味/有点甜"), or shows a reference sample and asks for "像这个但不一样的". Also trigger when the user produces their own attempt mid-conversation and is clearly seeking critique rather than approval. This skill turns vague taste signals into a structured convergence process — do not skip it just because the task "feels like simple writing"; the discipline below is exactly what separates useful iteration from frustrating ping-pong.
---

# taste-iteration

When the user wants to co-create something taste-driven (a sentence, a name, a tone, a slogan, a design choice), they almost always have strong preferences but cannot articulate them upfront. They learn what they want by **reacting to candidates**. Your job is **撒坐标 → 收坐标 → 收敛**: spread candidates across the possibility space, read their feedback as direction vectors, and tighten the space each round.

## Core principle

Do not try to produce the final answer in one shot. Treat each round as a probe that narrows the space. The user's feedback — especially negative feedback — is the most information-dense signal you will get. Extract it carefully; do not waste it.

## The six-step loop

### 1. Reverse-engineer the reference before imitating

If the user gives a reference sample, **拆解它为什么成立** before producing anything. Hit multiple dimensions:

- 句式骨架 / 语法结构
- 语义机制(明面说什么 vs 暗面说什么)
- 关键意象 / 关键词的选择理由
- 节奏 / 字数 / 对称
- 隐藏的逻辑陷阱(tautology、反讽、错位、留白等)

This breakdown becomes the **特征清单** for all subsequent generation. Skipping it means every round is blind imitation and the user will get frustrated faster than you can iterate.

Domain mapping:
- 起名 → 拆音节数 / 词根 / 语义距离 / 行业暗示
- 写文案 → 拆人称 / 长度 / 价值锚点 / 拒绝什么
- API/代码风格 → 拆抽象层次 / 命名密度 / 错误处理风格 / 注释策略
- 视觉/UI 调性 → 拆色彩温度 / 留白比例 / 对齐方式 / 字重对比

### 2. First batch is for "测坐标", not for "交付"

Generate 3–5 candidates that **deliberately span different directions**. Rules:

- Each candidate must walk a **noticeably different axis** — not 近义词式 variants of the same骨架
- Tag each candidate with which axis it explores ("对偶/起兴/比喻/反问")
- Every candidate must be a sincere attempt — never fill a slot with a weak option just to make up the count
- End with an explicit recommendation + reason ("我推 A,因为 X")

The recommendation matters: it gives the user a **handle to push back against**. Without it, the user has to do all the discrimination work themselves.

The goal of this round is not to land the right answer. It is to let the user **point with their rejection**.

### 3. Read negative feedback at the mechanism level, not the literal level

This is the most error-prone step in the loop. The user's "不要 X" is almost never literal — diagnose the mechanism behind the symptom.

| User says | Wrong reading | Correct reading |
|---|---|---|
| "太侧重老去" | Avoid the words 老/告别 | Don't anchor emotion at "终点感"; anchor at 重复/此刻/选择 |
| "有点装" | Add more flourish | Flourish 不缺,缺克制;"装" 通常 = 自我中心式抒情("我看的不是 X 是 Y") |
| "不够文艺" | Add ornate language | Distinguish 文艺 from ornate — user wants craft, not decoration |
| "不喜欢 2 4 7" | Just avoid those three | Find the **共同病灶** of 2/4/7 — the病灶 is the real signal |
| "太甜了" | Make it bitter | Probably means too many soft words clustered ("缓缓+也+不舍得"); cut one, don't reverse polarity |

**Operating principle**: 用户给你的是症状,你要诊断的是病因。停在症状层,下一轮还会犯同病。

### 4. When positive feedback appears, extract the axis immediately

The moment the user says "我喜欢 1 和 8,不喜欢 2 4 7" — the discriminating axis is right there. Do not ask the user to articulate it. **You** do the comparative analysis.

Operating procedure:
1. List the features of the "liked" group and the "disliked" group
2. Find the one (or two) features that **discriminate** between them
3. Say the axis out loud to the user in one sentence, for them to校对
4. Make that axis a **hard constraint** for the next batch

Why say it out loud:
- Lets the user correct your reading (they often will)
- Makes your next batch **explainable** — the user can predict what's coming and trust drops massively

### 5. Batch generation: differentiate on骨架, not on words

Once the axis is locked, produce 8–10 candidates, each with a **different syntactic / structural骨架**, all consistent with the axis.

- Bad: 10 个近义词式变体 of the same骨架
- Good: 起兴 / 对偶 / 拟人 / 主客颠倒 / 时间切片 / 比喻 / 否定让步 / 排比 — each labeled, each walking its own structural axis

The user is selecting from a **structural menu**, not a thesaurus. This is the round where convergence usually happens.

### 6. When the user produces their own attempt, switch to editor mode

The single most important behavioral switch in this skill: **the moment the user writes their own version, stop being a generator and become an editor.**

Editor stance = three-piece set, all three required:

1. **Specific praise** — name the exact word/phrase that works **and why** ("'赶路' 给画面一个具体语境,比抽象的'快'强")
2. **Specific pushback with reasoning** — name the weak point and explain the mechanism ("'缓缓' + '也' + '不舍得' 三个柔软词连用,整句偏甜")
3. **2–3 alternative revisions, not 1 prescribed answer** — comparison is where learning happens; let the user pick

Do not just clap. Specific praise + specific pushback + alternatives is how you upgrade the user, not just the artifact. A user who only gets applause stops growing; a user who gets prescribed single answers stops choosing.

## Meta-rules (apply across all six steps)

- **诊断 > 模仿**: Always crack the mechanism before producing.
- **骨架 > 形容词**: Diversity comes from structure, not synonyms.
- **负反馈拆机制 > 字面理解**: "不要 X" is almost never literal.
- **正反馈做对比 > 让用户解释**: The user has no obligation to know why they like something. You extract the axis.
- **编辑姿态 > 鼓掌姿态**: When the user produces, react like an editor — three-piece set.

## When NOT to use this skill

Skip this skill when:
- The user gave precise specifications upfront ("写一封 200 字的离职信,要求 A/B/C 三点")
- The output is objectively verifiable (compiling code, correct math, factual answer)
- The user explicitly said "just give me one, don't iterate"
- The task is purely informational, not creative

Use this skill when the convergence is **taste-driven** and the user is willing to iterate. If unsure, lean toward using it — the worst case is one extra round of structured candidates, which is rarely wasted.
