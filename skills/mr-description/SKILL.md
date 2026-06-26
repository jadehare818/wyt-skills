---
name: mr-description
description: Use when authoring a description / body for a merge request, pull request, or change list. Produces a structured Markdown description with English section titles and Chinese body content — required sections What / Why / Key Changes / Safety, plus optional Design Decisions / Verification / Breaking Changes. Platform-agnostic: works for Codebase MRs, GitHub PRs, GitLab MRs, Gerrit CLs, etc.
---

# mr-description

Use this skill whenever you need to write the **description / body** of a code change submission — merge request (Codebase, GitLab), pull request (GitHub), change list (Gerrit), patch cover letter, etc. This skill is about the **content** of the description; it does not invoke any host's API. Pair it with a host-specific creation skill (e.g. `create-mr` for Codebase, `gh pr create` for GitHub) that decides where the description gets posted.

## Triggering conditions

Invoke this skill when:

- A host-specific creation skill (e.g. `create-mr`) is preparing an MR / PR body
- The user asks "帮我写一下 MR 描述" / "draft the PR body" / "写一下 commit cover letter" without yet pushing
- You're updating an existing MR / PR description after follow-up commits

If the user only asks to create / open the MR / PR and you're about to compose the description as part of that, **always go through this skill first** to produce the body, then hand the resulting text to the platform tool.

## Output contract — the description must follow this template

**Section titles are English. Body content is Chinese.** This is non-negotiable.

Write the output to a temp file (`/tmp/<slug>-mr-desc.md`) — multi-line Markdown with backticks and Chinese punctuation does not survive shell quoting reliably. The caller skill / command then reads from that file.

### Required sections (in this order, do not reorder)

```markdown
## What

<本次改动做了什么的一段话总结，中文。说清楚改动覆盖的范围和目标，从「产品 / 用户能看到的功能」视角描述。不要重复 commit 列表，那是 Key Changes 的事。1–3 段足矣。>

## Why

<为什么做这件事。中文。可枚举多条：
1. 业务 / 技术上的根因（"老 SDK 拉闸"、"用户体验回归"、"性能瓶颈在 X"…）
2. 替代方案为何不采用
3. 隐含约束（"上游接口要求 source 字段必传"、"调用方默认 30s 太短"…）

每一条尽量是一句独立的、能反驳的事实陈述，不是"应该 / 也许"。>

## Key Changes

<按"被修改的文件"组织的 bullet list。中文。
- `path/to/file.ext` — 这里做了什么、为什么；如果是新增函数 / 类 / 字段，写出名字
- `path/to/other.ext` — ...

对复杂改动允许分小节：
### 核心逻辑
- ...
### 测试
- ...
### 配置
- ...

不要写成"这里改了一些代码"——具体到改的是哪个函数 / 哪个字段 / 行数。>

## Safety

<回归风险与缓解措施。中文。每一条都应该是可被读者反驳或验证的事实，不是空话。

应当覆盖（按需选择，不要硬凑）：
- 兼容性：老的 caller / 老的数据格式 / 老的配置是否仍能工作？以什么机制保证？
- 错误处理：异常路径有没有退化？fallback 存在吗？
- 测试覆盖：哪些路径被新单测覆盖？现有测试是否仍 pass？
- 风控：监控 / 告警 / 回滚预案是什么？
- 已知未解的风险：明确写"未阻塞 (non-blocking) / 后续单独跟进"，不要藏起来>
```

### Optional sections (include only when they have real content)

Add these **only** if there's substance — do not write "无" / "N/A" / "不适用" to pad. An empty section is worse than no section.

- `## Design Decisions` — when 2+ approaches were considered and you picked one; explain **why the rejected approaches were rejected**, not just why the chosen one was chosen. Empty Design Decisions ≠ "no design decisions"; it means "no decisions worth explaining to a reviewer".
- `## Verification` — manual / e2e validation steps that go beyond unit tests; if you ran a real command and saw real output, paste a representative line or two.
- `## Breaking Changes` — when ANY external interface changes meaning, signature, default value, or removed surface. Spell out the migration path for callers. If you have to think "is this a breaking change?", err toward including this section.
- `## Nacos Config` / `## Paired Config (mlp-terraform)` / `## Database Migration` / similar project-specific sections — include if the target project conventionally calls them out; omit entirely otherwise.

### Footer

Always include verbatim at the very end:

```markdown
🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

## Drafting workflow

1. **Read recent commits** on the branch (`git log <target>..HEAD --stat`) so What / Why / Key Changes line up with what's actually shipping. Description is not free-form storytelling; it has to match the diff.
2. **Identify the diff's footprint** — which files, what type of change (new / modified / deleted, behavior change vs refactor). Drives Key Changes.
3. **For Why**: ask yourself "if a reviewer doesn't believe me, what would I cite?". That's the evidence each Why bullet should carry.
4. **For Safety**: pretend you're a hostile reviewer hunting for regressions. Write down the questions you'd ask and how the change answers each one.
5. **Drop optional sections that have nothing real to say**. Empty section is a vendor's bullet point, not engineering communication.
6. **Write to `/tmp/<slug>-mr-desc.md`**. Caller is responsible for picking it up.

## Style notes

- Bullets prefer "事实陈述句" over "动词短语"; e.g. write "`refresh()` 现在在 sts 过期时同步调用 `auth.get_credentials`" rather than "更新 `refresh()`"。
- 涉及英文术语首次出现时附上原词，例如 "回归 (regression) 风险"、"覆盖 (override)"；后续可直接用中文。
- 文件路径、函数名、配置 key 一律 backtick 包裹。
- 不要写 ETA / Sprint 编号 / 个人名字 — 这些信息在 issue / ticket 里维护，不属于 MR 描述。

## Reference example

[maas-backend!15768](https://code.byted.org/machinelearning/maas-backend/merge_requests/15768) — real-world MR that follows this template; use it as a calibration anchor for tone and depth.

## Anti-patterns (do NOT do)

- 把整个 commit message 复制粘贴当描述
- 用 "Improve X" / "Fix Y" 这种动词起一个 bullet 然后没下文
- 给可选段写 "无" / "N/A" / "暂无" 来凑齐结构
- 写"细节见代码" / "self-explanatory" / "trivial change" 等推卸性语句
- 把 Why 和 What 混在一起——Why 解释**动机**，What 描述**结果**；一段话同时讲两件事会让人读不进去
- 章节标题用中文（"## 改动内容"）——必须英文
