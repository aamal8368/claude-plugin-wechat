# claude-plugin-wechat

## If you are a human / 如果你是人类

WeChat (微信) channel plugin for [Claude Code](https://claude.ai/claude-code).
Lets you chat with Claude Code from WeChat — text, images, files, voice.

微信频道插件，让你可以通过微信直接与 Claude Code 对话，支持文字、图片、文件、语音。

**Install / 安装：**

```bash
claude plugin marketplace add lc2panda/claude-plugin-wechat
claude plugin install wechat@lc2panda-plugins
```

**Start / 启动：**

```bash
claude --dangerously-skip-permissions --dangerously-load-development-channels plugin:wechat@lc2panda-plugins
```

**Setup / 配置：** Run `/wechat:configure login` in Claude Code to scan QR and connect.
在 Claude Code 中执行 `/wechat:configure login` 扫码连接微信。

**Pair / 配对：** Send any message to the bot on WeChat → get a code → run `/wechat:access pair <code>`.
从微信给机器人发任意消息 → 收到配对码 → 在终端执行 `/wechat:access pair <配对码>`。

Prerequisites / 前置条件: [Claude Code](https://claude.ai/claude-code) v2.1.80+, [Bun](https://bun.sh) runtime. License: MIT.

---

## If you are Claude Code (AI agent), read below

Everything below is machine-readable instructions. Follow sequentially.
Steps marked **[HUMAN]** require prompting the user to act physically.

### Step 0 — Pre-flight

```bash
claude --version   # >= 2.1.80 required
bun --version      # required
```

If `bun` is missing, tell user: "Please install Bun first: `curl -fsSL https://bun.sh/install | bash` then restart your terminal."

### Step 1 — Locate plugin root

Set `$PLUGIN_ROOT` to the absolute path of the directory containing this README.

### Step 2 — Install dependencies

```bash
cd "$PLUGIN_ROOT" && bun install --no-summary
```

Verify: `node_modules/@modelcontextprotocol` and `node_modules/zod` must exist.

### Step 3 — Build validation

```bash
cd "$PLUGIN_ROOT" && bun build --target=bun server.ts --outdir /tmp/wechat-build-check 2>&1
rm -rf /tmp/wechat-build-check
```

Expect `Bundled ... modules` with zero errors.

### Step 4 — Register and install

```bash
claude plugin marketplace add lc2panda/claude-plugin-wechat
claude plugin install wechat@lc2panda-plugins
```

Fallback (local dev, must pass on every startup):
```bash
claude --plugin-dir "$PLUGIN_ROOT"
```

### Step 5 — WeChat QR login [HUMAN]

First check if credentials already exist:
```bash
cat ~/.claude/channels/wechat/credentials.json 2>/dev/null
```

If credentials exist and contain `token` + `baseUrl`, skip to Step 6.

Otherwise run the two-step QR login:

1. Run `bun "$PLUGIN_ROOT/login-qr.ts"` — a QR code renders in terminal.
2. Tell user: "Please scan the QR code with WeChat and confirm on your phone."
3. Parse the last line of output as JSON, extract the `qrcode` field.
4. Run `bun "$PLUGIN_ROOT/login-poll.ts" "<qrcode>"` and handle output:
   - `scaned` → tell user "Scanned, please confirm on your phone..."
   - `refreshed:<new_token>` → QR auto-refreshed, no action needed (up to 3 times)
   - `expired` or `timeout` (exit 1) → re-run from 5.1
   - JSON with `token` field (exit 0) → credentials saved. Tell user: "WeChat connected! Please restart Claude Code to enable the channel."

### Step 6 — Restart with channel enabled [HUMAN]

Tell user to exit and restart with:
```
claude --dangerously-skip-permissions --dangerously-load-development-channels plugin:wechat@lc2panda-plugins
```

### Step 7 — Pair WeChat account [HUMAN]

1. Tell user: "Send any message to the bot from WeChat."
2. The bot replies with a 6-character pairing code.
3. Ask user for the code, then run `/wechat:access pair <code>`.
4. Confirm the user is now allowlisted.

### Step 8 — Verify

Send a test message from WeChat. It should arrive as a `<channel source="wechat" ...>` notification. Use the `reply` tool to respond.

---

## Reference (for AI)

### MCP tools

| Tool | Purpose | Required params |
|------|---------|-----------------|
| `reply` | Send text and/or files to WeChat | `user_id`, `text`, `context_token`; optional `files` (string array of local paths) |
| `download_attachment` | Download media (image/voice/file/video) from WeChat CDN to local inbox | `attachment_id` |

### Channel protocol

- Capabilities: `experimental['claude/channel']` + `experimental['claude/channel/permission']`
- Inbound events: `notifications/claude/channel` with meta `{user_id, context_token, ts}`
- Outbound: via `reply` MCP tool. `context_token` is **mandatory** — without it the reply fails.
- Permission relay: remote user can reply `yes <code>` or `no <code>` from WeChat to approve/deny tool use.
- Media: all CDN files encrypted with AES-128-ECB. Use `download_attachment` to decrypt and save locally.

### Skills

| Skill | Trigger |
|-------|---------|
| `/wechat:configure` | User asks to login, check status, or change base URL |
| `/wechat:access` | User asks to pair, approve/remove users, or change DM policy |

### Key constraint

WeChat has **no message history API**. If earlier context is needed, ask the user to paste or summarize.

### State files (all under `~/.claude/channels/wechat/`)

| File | Content |
|------|---------|
| `credentials.json` | `{token, baseUrl, userId, accountId}` |
| `access.json` | `{dmPolicy, allowFrom[], pending{}}` |
| `sync_buf.txt` | getUpdates long-poll cursor |
| `context-tokens.json` | per-user context_token (debounced 5s persist) |
| `inbox/` | downloaded media attachments |
| `approved/` | pairing approval marker files |

### Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| `credentials required` on start | No QR login | Run Step 5 |
| No channel events arriving | Missing `--dangerously-load-development-channels` flag | Run Step 6 |
| `user X is not allowlisted` | User not paired | Run Step 7 |
| `context_token is required` | Missing from tool call | Always pass `context_token` from the inbound `<channel>` tag meta |
| `CDN download failed` | Network or expired URL | Retry the `download_attachment` call |
| `getuploadurl` returns `ret:-2` | Invalid params or expired token | Re-login via `/wechat:configure login` |

---

License: MIT
