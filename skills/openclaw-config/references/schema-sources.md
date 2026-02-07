# OpenClaw Config: Schema Sources

这份技能的目标是避免“字段写错/类型不对/约束没满足”导致 OpenClaw Gateway 拒绝启动或行为异常。
配置是 **JSON5**，并且大部分配置段是 **strict**（未知 key 直接报错）。

参考源码版本: `openclaw/openclaw@875324e`（克隆时间: 2026-02-07）。
不同版本可能有字段变更，**以你正在运行的 OpenClaw 版本为准**。

## 优先级: 如何确认字段是否支持

1. Gateway 正在运行时（最推荐）
   - 直接获取 JSON Schema:
     - `openclaw gateway call config.schema --params '{}'`
   - 你可以用 `jq`/搜索来确认字段路径是否存在，再决定写入哪些 key。

2. Gateway 未运行 / 需要看源码约束
   - 克隆源码:
     - `git clone https://github.com/openclaw/openclaw.git`
   - 关键 schema 文件:
     - 根 schema: `src/config/zod-schema.ts`（`OpenClawSchema`）
     - `$include` 语义: `src/config/includes.ts`
     - agents/tools: `src/config/zod-schema.agents.ts`, `src/config/zod-schema.agent-defaults.ts`, `src/config/zod-schema.agent-runtime.ts`
     - models: `src/config/zod-schema.core.ts`（`ModelsConfigSchema`）
     - channels: `src/config/zod-schema.providers.ts`, `src/config/zod-schema.providers-core.ts`, `src/config/zod-schema.providers-whatsapp.ts`
     - session/messages/commands: `src/config/zod-schema.session.ts`
     - approvals: `src/config/zod-schema.approvals.ts`
   - 仓库内配置文档（带大量例子）:
     - `docs/gateway/configuration.md`

## 快速定位技巧（不要“猜 key”）

在 openclaw 源码根目录运行:

```bash
rg -n "export const OpenClawSchema" src/config/zod-schema.ts
rg -n "\\bgateway:\\s*z" src/config/zod-schema.ts
rg -n "\\bskills:\\s*z" src/config/zod-schema.ts
rg -n "\\bplugins:\\s*z" src/config/zod-schema.ts

rg -n "export const ChannelsSchema" src/config/zod-schema.providers.ts
rg -n "DiscordConfigSchema|TelegramConfigSchema|SlackConfigSchema" src/config/zod-schema.providers-core.ts

rg -n "export const ModelsConfigSchema" src/config/zod-schema.core.ts
rg -n "export const ToolsSchema" src/config/zod-schema.agent-runtime.ts
```

## 读懂校验错误的方式

`openclaw doctor` 报错通常包含:
- `path`: 出错字段路径（最重要）
- `message`: 错误原因（类型不匹配、未知 key、缺失必填、跨字段约束失败等）

处理策略:
- **未知 key**: 说明该字段在 schema 中不存在或拼写不对。去 schema 文件里确认正确字段名。
- **类型不匹配**: 按 schema 改成正确类型（number/string/boolean/object/array）。
- **约束失败（superRefine）**: 按报错 message 满足关联字段（例如某些 channel 的 `dmPolicy="open"` 需要 `allowFrom` 包含 `"*"`）。

