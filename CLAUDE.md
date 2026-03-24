# CLAUDE.md — claude-channel-weixin 项目记忆

> 此项目的任何功能、架构更新，必须在结束后同步更新相关文档。这是我们契约的一部分。

---

## 0. 时间真实性校验

| 项目 | 值 |
|------|-----|
| 校验发起 | 2026-03-24 10:10:37 +08:00 |
| 校验完成 | 2026-03-24 10:10:45 +08:00 |
| 本机系统时间 | 2026-03-24 10:10:37 +08:00 (Asia/Singapore, +08:00) |
| 时间源 1 | Baidu HTTPS Date Header → `Tue, 24 Mar 2026 02:10:43 GMT` = 10:10:43 +08:00 |
| 时间源 2 | Google HTTPS Date Header → `Tue, 24 Mar 2026 02:10:45 GMT` = 10:10:45 +08:00 |
| 最大偏差 | 8 秒（阈值 100 秒） |
| **判定** | **通过 ✓** |
| 备注 | 此时间锚点用于后续所有检索记录与日志 |

---

## 1. 项目概览

**项目名称**：claude-channel-weixin（微信频道插件）
**版本**：plugin v0.4.0 / package v0.1.0
**许可证**：MIT
**运行时**：Bun
**核心功能**：基于腾讯 iLink Bot API 的微信消息桥接插件，使 Claude Code 可直接收发微信消息。

### 架构

```
微信用户 → WeChat App → iLink Bot API (ilinkai.weixin.qq.com)
                              ↕ HTTP Long-Poll
                     server.ts (MCP Server, 本地运行)
                              ↕ MCP Protocol (stdio)
                         Claude Code Session
```

### 文件清单

| 文件 | 职责 | 行数 |
|------|------|------|
| `server.ts` | 主 MCP 服务器：长轮询收消息、发消息、访问控制 | ~518 |
| `login-qr.ts` | 登录步骤1：获取并显示终端 QR 码 | ~39 |
| `login-poll.ts` | 登录步骤2：轮询扫码状态、保存凭据 | ~115 |
| `package.json` | 依赖声明（MCP SDK + qrcode-terminal） | ~14 |
| `.mcp.json` | MCP 服务器启动配置 | ~7 |
| `.claude-plugin/plugin.json` | Claude Code 插件元数据 | ~7 |
| `skills/configure/SKILL.md` | /weixin:configure 技能定义 | — |
| `skills/access/SKILL.md` | /weixin:access 技能定义 | — |
| `README.md` | 用户文档 | ~65 |

### 依赖

- `@modelcontextprotocol/sdk` ^1.0.0 — MCP 服务器框架
- `qrcode-terminal` ^0.12.0 — 终端 QR 码渲染

### 状态存储

所有运行时状态位于 `~/.claude/channels/weixin/`：
- `credentials.json` — bot_token + baseUrl + userId + accountId
- `access.json` — dmPolicy / allowFrom / pending（配对码）
- `sync_buf.txt` — getUpdates 游标
- `approved/` — 新配对用户标记目录

---

## 2. 证据清单（联网检索记录）

### 议题：腾讯 iLink Bot API 技术规范与合法性

**检索时间**：2026-03-24 10:10:45 +08:00

#### 来源 1（权威社区技术文档）
- **URL**：https://github.com/hao-ji-xing/openclaw-weixin/blob/main/weixin-bot-api.md
- **类型**：开源社区逆向整理的完整 API 文档
- **发布日期**：2026-03 (活跃维护)
- **摘要**：完整记录了 iLink Bot API 的 7 个端点、认证流程、消息结构、AES-128-ECB 媒体加密、context_token 机制
- **采纳性**：✅ 采纳 — 最完整的 API 技术参考，与本项目实现完全吻合

#### 来源 2（科技媒体报道 — TechBriefly）
- **URL**：https://techbriefly.com/2026/03/23/tencent-launches-clawbot-linking-wechat-to-openclaw/
- **类型**：国际科技媒体
- **发布日期**：2026-03-23
- **摘要**：腾讯于 2026-03-23 正式发布 ClawBot 插件，将微信接入 OpenClaw AI 代理框架；已在 QQ 和企业微信中先行集成
- **采纳性**：✅ 采纳 — 确认 iLink Bot API 是腾讯官方合法开放接口

#### 来源 3（权威媒体 — 南华早报 SCMP）
- **URL**：https://www.scmp.com/tech/article/3347590/tencent-adds-clawbot-plug-wechat-amid-openclaw-boom-and-privacy-warnings
- **类型**：国际权威媒体
- **发布日期**：2026-03-23
- **摘要**：腾讯总裁陶贤淇确认隐私保护是微信代理开发的关键挑战；中国网络安全协会建议仅在专用设备运行 OpenClaw；ClawBot 面向 10 亿+月活用户
- **采纳性**：✅ 采纳 — 确认合法性与隐私风险提示

#### 来源 4（53AI 技术社区）
- **URL**：https://www.53ai.com/news/Openclaw/2026032373016.html
- **类型**：中国 AI 技术社区
- **发布日期**：2026-03-23
- **摘要**：详述 Claude Code 集成方案，约 300 行代码通过 MCP Channel 桥接；官方 npm 包 `@tencent-weixin/openclaw-weixin` v1.0.2
- **采纳性**：✅ 采纳 — 确认集成架构与本项目一致

#### 来源 5（npm 官方仓库）
- **URL**：https://www.npmjs.com/package/@tencent-weixin/openclaw-weixin
- **类型**：npm 包注册表
- **版本**：1.0.2（2026-03-22 发布）
- **摘要**：腾讯微信官方 OpenClaw 插件包，含 CLI 安装工具
- **采纳性**：✅ 采纳 — 确认腾讯官方 npm 包存在

#### 来源 6（LINUX DO 社区）
- **URL**：https://linux.do/t/topic/1800355
- **类型**：开发者社区讨论
- **发布日期**：2026-03（403 无法访问全文）
- **摘要**：开发者 Johnixr 修改 ClawBot 以支持 Claude Code 的经验分享
- **采纳性**：⚠️ 部分采纳 — 标题确认方向，全文无法访问

### 本地已有实现

- **路径**：`/Users/panda/Downloads/download/claude-plugin-wechat/`
- **关联提交**：`5f28254` (初始) → `d870a09` (最新)
- **复用说明**：本项目即为 iLink Bot API 的 Claude Code MCP 桥接实现，与来源 4 描述的架构一致

### 结论

✅ **采用** — 腾讯 iLink Bot API 是 2026-03-23 官方发布的合法微信个人账号 Bot 接口。本项目正确实现了其核心协议（QR 登录 → 长轮询收消息 → context_token 回传发消息）。API 端点、认证方式、消息结构均与权威文档一致。

---

## 3. iLink Bot API 核心技术摘要

### API 端点（域名：`https://ilinkai.weixin.qq.com`）

| 端点 | 方法 | 功能 |
|------|------|------|
| `/ilink/bot/get_bot_qrcode?bot_type=3` | GET | 获取登录 QR 码 |
| `/ilink/bot/get_qrcode_status?qrcode=<token>` | GET | 轮询扫码状态 |
| `/ilink/bot/getupdates` | POST | 长轮询收消息（35s 超时） |
| `/ilink/bot/sendmessage` | POST | 发送消息 |
| `/ilink/bot/getuploadurl` | POST | 获取 CDN 预签名上传地址 |
| `/ilink/bot/getconfig` | POST | 获取 typing_ticket |
| `/ilink/bot/sendtyping` | POST | 发送"正在输入"状态 |

### 认证头

```
Content-Type: application/json
AuthorizationType: ilink_bot_token
Authorization: Bearer <bot_token>
X-WECHAT-UIN: base64(String(randomUint32()))  // 每次随机，防重放
```

### 消息结构

- **用户 ID 格式**：`xxx@im.wechat`（用户）/ `xxx@im.bot`（机器人）
- **消息类型**：1=文本, 2=图片, 3=语音, 4=文件, 5=视频
- **context_token**：每条消息必带，回复时必须原样回传
- **媒体加密**：AES-128-ECB，CDN 域名 `novac2c.cdn.weixin.qq.com`

### 关键限制

- 无历史消息拉取 API
- 速率限制未公开
- 目前 iOS 优先支持
- 一个 ClawBot 仅连接一个 OpenClaw 实例
- 群聊权限模糊

---

## 4. 本项目与官方实现的对比

| 维度 | 本项目实现 | 官方 API 规范 | 一致性 |
|------|-----------|-------------|--------|
| 登录流程 | QR获取 → 轮询确认 → 存凭据 | get_bot_qrcode → get_qrcode_status → bot_token | ✅ 一致 |
| 收消息 | getupdates 长轮询 + sync_buf | getupdates + get_updates_buf | ✅ 一致 |
| 发消息 | sendmessage + context_token | sendmessage + context_token | ✅ 一致 |
| 认证头 | Bearer + randomUIN | ilink_bot_token + random UIN | ✅ 一致 |
| 消息分块 | 2000 字符限制 | 微信文本限制 | ✅ 合理 |
| 媒体支持 | 仅文本（图片/语音/视频显示占位符） | 完整支持（AES-128-ECB 加解密） | ⚠️ 待扩展 |
| typing 状态 | 未实现 | getconfig + sendtyping | ⚠️ 待扩展 |
| CDN 上传 | 未实现 | getuploadurl + PUT | ⚠️ 待扩展 |

---

## 5. 冗余治理报告

检查结果：项目文件结构清晰，无同名/同责/高相似冗余文件。各文件职责单一明确。

---

## 6. 特例登记

（暂无）

---

## 7. 技巧区（Claude Code 集成）

- 计划模式：Shift+Tab 生成计划后再编码
- 测试：修改后运行 `bun server.ts` 验证启动
- 上下文管理：login-qr.ts 和 login-poll.ts 为独立脚本，可单独测试
