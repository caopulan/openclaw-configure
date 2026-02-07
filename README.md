# OpenClaw Configure

Community-friendly configuration helpers for [OpenClaw](https://github.com/openclaw/openclaw).

This repository currently ships a Codex skill that helps you edit and validate OpenClaw's
Gateway config (`openclaw.json`) safely, using a schema-first workflow.

## What's in this repo

- `skills/openclaw-config/`: Codex skill for editing OpenClaw config without breaking schema validation

## Why this exists

OpenClaw validates config strictly: unknown keys or wrong types can prevent the Gateway from starting.
The `openclaw-config` skill focuses on:

- Finding the authoritative schema (prefer `config.schema` from the running Gateway)
- Making minimal, targeted edits (prefer `openclaw config set|get|unset`)
- Validating changes with `openclaw doctor`
- Avoiding common pitfalls (`.strict()` objects, `$include` merge rules, channel/provider constraints)

## Quick start (install the Codex skill)

```bash
git clone git@github.com:caopulan/openclaw-configure.git
cd openclaw-configure

mkdir -p ~/.codex/skills/openclaw-config
rsync -a skills/openclaw-config/ ~/.codex/skills/openclaw-config/
```

Then use the skill by name in Codex: `openclaw-config`.

## Notes

- Do not commit secrets. This repo ignores `.env`, key files, and common credential formats via `.gitignore`.
- OpenClaw config is JSON5 (comments + trailing commas).
- Official config docs live in the upstream repo: `openclaw/openclaw` (`docs/gateway/configuration.md`).
