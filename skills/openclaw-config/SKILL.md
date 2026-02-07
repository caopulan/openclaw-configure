---
name: openclaw-config
description: OpenClaw 配置(openclaw.json/JSON5)修改与校验指南。用于新增/调整 OpenClaw Gateway 配置字段（例如 gateway.*, agents.*, models.*, channels.*, tools.*, skills.*, plugins.*、$include），或排查 openclaw doctor/config 校验报错；目标是避免未知字段/类型不匹配导致 Gateway 拒绝启动、运行异常或安全策略失效。
---

# OpenClaw Config

## Overview

以“schema 为准”的方式安全修改 `~/.openclaw/openclaw.json`（或 `OPENCLAW_CONFIG_PATH` 指定的配置），并在提交前后做严格校验，避免因字段不合法引入 bug。

## Workflow (Safe Edit)

1. **确认正在使用的配置文件路径**

- 优先级: `OPENCLAW_CONFIG_PATH` > `OPENCLAW_STATE_DIR/openclaw.json` > `~/.openclaw/openclaw.json`
- 配置文件是 **JSON5**（允许注释、尾逗号）。

2. **先拿到“权威 schema”，不要猜字段**

- Gateway 正在跑: 用 `openclaw gateway call config.schema --params '{}'` 拿 JSON Schema（最贴合当前版本）。
- 否则: 以 `openclaw/openclaw` 源码为准，核心文件:
  - `src/config/zod-schema.ts`（`OpenClawSchema`，根字段 + `gateway`/`skills`/`plugins` 等）
  - `src/config/zod-schema.*.ts`（各子模块，比如 channels/providers/models/agent）
  - `docs/gateway/configuration.md`（仓库内配置文档与示例）

3. **以最小变更方式写入配置**

- 小改动优先: `openclaw config set|get|unset`（点路径/括号路径）。
- Gateway 在线且希望“写入 + 校验 + 重启”一条龙: 用 RPC `config.patch`（局部 merge patch）或 `config.apply`（替换整个配置，谨慎）。
- 复杂配置建议用 `$include` 拆分（见下文）。

4. **严格校验**

- 运行 `openclaw doctor` 读清楚每条 issue 的 `path` 和 `message` 再改。
- **不要在未获用户明确同意时**执行 `openclaw doctor --fix/--yes`（会写入配置/状态文件）。

## Guardrails (Avoid Schema Bugs)

- **绝大多数对象是 strict**: 多数配置段用 `.strict()`，出现未知 key 会直接导致校验失败，Gateway 拒绝启动。
- `channels` 段是 `.passthrough()`: 允许扩展渠道（matrix/zalo/nostr 等）挂载自定义 key，但内层 provider 配置通常仍是 strict。
- `env` 段是 `.catchall(z.string())`: 允许把环境变量直接写到 `env` 下（string 值），同时也支持 `env.vars`。
- **敏感信息**: 尽量用环境变量/凭据文件，不要把长期 token/API key 明文写进 `openclaw.json`。

## $include (Modular Config)

`$include` 在 schema 校验前解析，用于把配置拆成多个 JSON5 文件:

- 支持 `"$include": "./base.json5"` 或数组 `"$include": ["./a.json5", "./b.json5"]`
- 相对路径以“当前配置文件所在目录”为基准解析
- 深度 merge 规则（来自源码实现）:
  - object: 递归合并
  - array: **拼接**（不是替换）
  - primitive: 后者覆盖前者
- 与 sibling key 共存时: sibling key 会覆盖 include 内容里的同名字段
- 限制: 最多 10 层 include；检测循环 include 并报错

## Common Recipes (Examples)

1. 设置默认 workspace

```bash
openclaw config set agents.defaults.workspace '"~/.openclaw/workspace"' --json
openclaw doctor
```

2. 修改 Gateway 端口

```bash
openclaw config set gateway.port 18789 --json
openclaw doctor
```

3. 拆分配置（示例）

```json5
// ~/.openclaw/openclaw.json
{
  "$include": ["./gateway.json5", "./channels/telegram.json5"],
}
```

4. Telegram 开放 DM（必须显式允许来源）

> 约束来自 schema：当 `dmPolicy="open"` 时，`allowFrom` 必须包含 `"*"`.

```bash
openclaw config set channels.telegram.dmPolicy '"open"' --json
openclaw config set channels.telegram.allowFrom '["*"]' --json
openclaw doctor
```

5. Discord 启用与 token（config 或环境变量回退）

```bash
# 方式 A: 写入 config
openclaw config set channels.discord.token '"YOUR_DISCORD_BOT_TOKEN"' --json

# 方式 B: 使用 env var 回退（仍建议至少存在 channels.discord 配置段）
# export DISCORD_BOT_TOKEN="..."

openclaw doctor
```

6. 开启 web_search（Brave / Perplexity）

```bash
openclaw config set tools.web.search.enabled true --json
openclaw config set tools.web.search.provider '"brave"' --json

# 推荐: 通过环境变量提供密钥（或写入 tools.web.search.apiKey）
# export BRAVE_API_KEY="..."

openclaw doctor
```

## Resources

需要字段索引/源码定位时再加载:

- `references/openclaw-config-fields.md`（根字段索引 + 关键段落的字段清单与来源）
- `references/schema-sources.md`（如何从 openclaw 仓库定位 schema 与约束）
- `scripts/openclaw-config-check.sh`（快速定位 config 路径 + 运行 doctor）
