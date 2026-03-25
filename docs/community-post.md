# claude-plugin-wechat — 用微信遥控你的 Claude Code

开源插件，两行命令安装，微信扫码即用。

## 能做什么

- 📝 **文字对话** — 微信发消息，Claude Code 直接收到并回复
- 🎤 **语音遥控** — 按住说话，自动转文字，零打字门槛
- 🖼️ **图片双向** — 发截图让 Claude 分析，Claude 也能给你发图
- 📁 **文件传输** — 双向传文件，34 种格式自动识别（图片/视频/文档）
- 🎬 **视频接收** — 接收并 AES 解密保存
- 🔐 **远程审批** — Claude 要执行敏感操作？微信回复 yes/no
- 💬 **引用回复** — 引用之前的消息继续聊
- 🐛 **Debug** — `/echo` 测延迟，`/toggle-debug` 看诊断

## 特色

**双模式，不挑认证方式：**
- claude.ai 用户 → Channel 模式（全功能含权限中继）
- API Key 用户 → Agent SDK 模式（支持任何 Provider）

**极简安装：**
```bash
claude plugin marketplace add lc2panda/claude-plugin-wechat
claude plugin install wechat@lc2panda-plugins
```

**不需要：** OpenClaw、Docker、公网 IP、服务器、图床

**安全设计：** 配对码认证 + 白名单 + 不暴露端口 + 禁止发送凭据

**与腾讯官方 100% 功能对齐：** 媒体加解密、SILK 语音转码、CDN 重试、Session 过期检测，逐项对标 `@tencent-weixin/openclaw-weixin` 源码

## 微信接 AI 方案怎么选？

目前社区里微信接 AI 的开源方案不少，各有侧重，简单梳理下方便大家按需选择：

| | claude-plugin-wechat（本项目） | wechat-acp | cc-connect | weixin-agent-sdk |
|---|---|---|---|---|
| **定位** | Claude Code 专属微信插件 | 通用 ACP 协议微信桥接 | 多平台多 Agent 桥接 | 通用 Agent SDK |
| **支持的 Agent** | Claude Code | 6 种（copilot/claude/gemini/qwen/codex/opencode） | 7+ 种 | 自定义 |
| **通信协议** | MCP Channel + Agent SDK 双模式 | ACP (JSON-RPC over stdio) | stdin/stdout pipe | ACP |
| **安装方式** | 2 行命令（Claude Code 插件） | npx 一行启动 | 配置文件 | 需写代码 |
| **认证要求** | claude.ai 或 API Key 均可 | 取决于 Agent | 取决于 Agent | 取决于 Agent |
| **媒体支持** | 全类型 5 种 34 格式 + AES 加解密 | 文字为主 | 语音/图片 | 基础 |
| **语音转码** | ✅ SILK→WAV | ❌ | ✅ STT | ❌ |
| **远程权限审批** | ✅（Channel 模式） | ❌（自动批准） | ❌ | ❌ |
| **多轮对话** | ✅ 持久 session | ✅ 每用户独立 session | ✅ | ✅ |
| **安全模型** | 配对码 + 白名单 + 无入站端口 | 无 | 依赖平台 | 自建 |
| **需要公网 IP** | ❌ | ❌ | 部分需要 | ❌ |

**怎么选：**
- **只用 Claude Code** → 选本项目，功能最全、媒体最强、安全最好
- **想在多个 Agent 间切换**（copilot/gemini/codex/qwen）→ 选 wechat-acp，一个桥接器接所有
- **需要接飞书/钉钉/Slack 等多平台** → 选 cc-connect
- **想自建定制 Agent** → 选 weixin-agent-sdk

各项目都是开源方案，解决的是同一个痛点：**让微信成为 AI 的遥控器**。侧重点不同，按需取用就好。

## 适合谁

用 Claude Code 的所有人 — 产品经理、运营、创作者、开发者。人不在电脑前，微信语音说一句就能让 Claude 干活。

## 链接

- **GitHub：** https://github.com/lc2panda/claude-plugin-wechat
- **企业微信版：** https://github.com/dividduang/claude-plugin-wecom

MIT 开源，欢迎 Star ⭐
