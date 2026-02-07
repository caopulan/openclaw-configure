# OpenClaw Config Field Index (openclaw.json)

参考源码版本: `openclaw/openclaw@875324e`（2026-02-07）。不同版本字段可能变化，建议优先用运行中的 Gateway 获取 `config.schema`。

配置文件: `~/.openclaw/openclaw.json`（JSON5）
- 可用 `OPENCLAW_CONFIG_PATH` 指定路径
- 可用 `$include` 拆分配置（语义见 `src/config/includes.ts`）

## Root Keys (OpenClawSchema)

根对象是 strict；除 `$include` 预处理外，未知 key 会导致校验失败。

- `meta`: 写入元信息（lastTouchedVersion/At）
- `env`: 环境变量导入与 sugar（允许 string catchall）
- `wizard`: 向导运行信息
- `diagnostics`: 诊断/otel/cacheTrace
- `logging`: 日志级别/输出/脱敏
- `update`: 更新通道与启动检查
- `browser`: Browser/CDP 配置
- `ui`: UI 外观与助理名/头像
- `auth`: 认证 profile/order/cooldowns
- `models`: 模型 providers/definitions
- `nodeHost`: Node Host 相关（目前包含 browserProxy）
- `agents`: agents.defaults + agents.list
- `tools`: 全局 tools policy + exec/web/media/links
- `bindings`: channel/account/peer 到 agent 的绑定规则
- `broadcast`: 广播策略与 peer->agentId 列表映射
- `audio`: 转写等音频配置
- `media`: 媒体文件处理（例如 preserveFilenames）
- `messages`: 消息行为/前缀等（见 session schema）
- `commands`: 聊天命令相关（见 session schema）
- `approvals`: 审批策略（见 approvals schema）
- `session`: 会话策略（见 session schema）
- `cron`: cron store/concurrency
- `hooks`: hooks server + gmail/internal mappings
- `web`: web socket/reconnect 等
- `channels`: 各渠道配置（whatsapp/telegram/discord/slack/...）
- `discovery`: mdns/wideArea
- `canvasHost`: Canvas Host
- `talk`: Talk/TTS 快捷配置
- `gateway`: Gateway 服务/鉴权/remote/tls/http endpoints/nodes
- `memory`: memory backend/citations/qmd
- `skills`: skills 加载/安装/entries
- `plugins`: plugins 加载/entries/installs

## gateway (常改字段)

来源: `src/config/zod-schema.ts` 的 `gateway` 段。

- `gateway.port`: number
- `gateway.mode`: `"local" | "remote"`
- `gateway.bind`: `"auto" | "lan" | "loopback" | "custom" | "tailnet"`
- `gateway.controlUi`:
  - `enabled`, `basePath`, `root`, `allowedOrigins`
  - `allowInsecureAuth`, `dangerouslyDisableDeviceAuth`
- `gateway.auth`:
  - `mode`: `"token" | "password"`
  - `token`, `password`, `allowTailscale`
- `gateway.trustedProxies`: string[]
- `gateway.tailscale`: `{ mode: "off" | "serve" | "funnel", resetOnExit }`
- `gateway.remote`:
  - `url`, `transport`: `"ssh" | "direct"`
  - `token`, `password`, `tlsFingerprint`
  - `sshTarget`, `sshIdentity`
- `gateway.reload`: `{ mode: "off" | "restart" | "hot" | "hybrid", debounceMs }`
- `gateway.tls`: `{ enabled, autoGenerate, certPath, keyPath, caPath }`
- `gateway.http.endpoints`:
  - `chatCompletions.enabled`
  - `responses.enabled`, `responses.maxBodyBytes`
  - `responses.files` / `responses.images`（allowUrl/allowedMimes/maxBytes/maxRedirects/timeoutMs 等）
- `gateway.nodes`:
  - `browser.mode`: `"auto" | "manual" | "off"`
  - `browser.node`: string
  - `allowCommands`, `denyCommands`: string[]

## skills / plugins (安装与条目)

来源: `src/config/zod-schema.ts` 的 `skills`/`plugins` 段。

`skills`:
- `skills.allowBundled`: string[]
- `skills.load`: `{ extraDirs, watch, watchDebounceMs }`
- `skills.install`: `{ preferBrew, nodeManager: "npm"|"pnpm"|"yarn"|"bun" }`
- `skills.entries.<id>`:
  - `enabled`: boolean
  - `apiKey`: string
  - `env`: record<string,string>
  - `config`: record<string,unknown>

`plugins`:
- `plugins.enabled`: boolean
- `plugins.allow` / `plugins.deny`: string[]
- `plugins.load.paths`: string[]
- `plugins.slots.memory`: string
- `plugins.entries.<id>`: `{ enabled, config }`
- `plugins.installs.<id>`:
  - `source`: `"npm" | "archive" | "path"`
  - `spec`, `sourcePath`, `installPath`, `version`, `installedAt`

## channels / models / agents / tools (建议按 schema 文件查)

这些段字段多且变化较快，建议“按 schema 文件定位”，不要手写猜测:

- `channels`: `src/config/zod-schema.providers.ts` + `src/config/zod-schema.providers-core.ts`
  - 注意: `channels` 段是 passthrough（允许扩展渠道 key）
  - 但每个 provider（telegram/discord/slack/...）对象通常是 strict
- `models`: `src/config/zod-schema.core.ts` 的 `ModelsConfigSchema`
- `agents`: `src/config/zod-schema.agents.ts` / `src/config/zod-schema.agent-defaults.ts` / `src/config/zod-schema.agent-runtime.ts`
- `tools`: `src/config/zod-schema.agent-runtime.ts` 的 `ToolsSchema`

