# wyt-skills

Personal Claude Code skills collection — lightweight workflows for everyday coding.

## Skills

- **plan-before-code** — Forces a short `story.md` + `architect-plan.md` under `.claude/plans/<slug>/` before any source-file edit, with a user-approval checkpoint. Keeps changes visible before code is written.
- **coding-brainstorm** — Opinionated multi-round brainstorm prerequisite for `plan-before-code`. Pushes the discussion from vague intent to grounded design before any planning artifact is written.
- **taste-iteration** — Iteratively refine an artifact (code / design / writing) against the user's "taste" by trying a deliberate change each round, eliciting reactions, and converging.
- **work-summary** — Produce a structured self-review of recent dev work by combining git history with interactive discussion. Triggered by "总结我的工作" / "summarize my work" / similar.
- **mr-description** — Author the description / body of a merge request, pull request, or change list. English section titles + Chinese body content; required sections What / Why / Key Changes / Safety. Platform-agnostic — pair with host-specific creation tools (e.g. `create-mr` for Codebase, `gh pr create` for GitHub).

## Install as a Claude Code plugin

Add this repo as a marketplace, then install the plugin:

```
/plugin marketplace add jadehare818/wyt-skills
/plugin install wyt-skills@wyt-skills
```

## Layout

```
.claude-plugin/
  plugin.json          # plugin manifest
  marketplace.json     # marketplace manifest (for /plugin marketplace add)
skills/
  <skill-name>/
    SKILL.md
```
