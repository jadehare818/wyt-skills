# wyt-skills

Personal Claude Code skills collection — lightweight workflows for everyday coding.

## Skills

- **plan-before-code** — Forces a short `story.md` + `architect-plan.md` under `.claude/plans/<slug>/` before any source-file edit, with a user-approval checkpoint. Keeps changes visible before code is written.

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
  plan-before-code/
    SKILL.md
```
