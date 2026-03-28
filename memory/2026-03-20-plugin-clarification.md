# Session: 2026-03-20 16:02:07 UTC

- **Session Key**: agent:main:feishu:default:direct:ou_d7505506d0ca45a509cc90322e839302
- **Session ID**: 53b58dc3-cb62-4708-b17d-89d5d8c93897
- **Source**: feishu

## Conversation Summary

user: System: [2026-03-19 18:07:22 GMT+8] Exec completed (calm-gla, code 0) :: /Users/weijian/.openclaw/workspace/skills/feishu-permission-setup-skill/.git /Users/weijian/.openclaw/workspace/skills/sow-generator/.git /Users/weijian/.openclaw/workspace/.git /…
System: [2026-03-20 14:16:34 GMT+8] Feishu[default] DM | 黄威健 (ou_d7505506d0ca45a509cc90322e839302) [msg:om_x100b54fa2adda0dcb3cd4e009a4f7db]

Conversation info (untrusted metadata):
```json
{
  "message_id": "om_x100b54fa2adda0dcb3cd4e009a4f7db",
  "sender_id": "ou_d7505506d0ca45a509cc90322e839302",
  "sender": "黄威健",
  "timestamp": "Fri 2026-03-20 14:16 GMT+8"
}
```

Sender (untrusted metadata):
```json
{
  "label": "黄威健 (ou_d7505506d0ca45a509cc90322e839302)",
  "id": "ou_d7505506d0ca45a509cc90322e839302",
  "name": "黄威健"
}
```

我记得之前不是有两个飞书插件的吗？一个飞书官方的，一个openclaw的，到底哪个是新的，现在保留的是哪个啊
assistant: 让我查一下当前安装的飞书插件情况。
assistant: 根据我查到的信息：

**现在只保留了一个飞书插件**：`@larksuite/openclaw-lark`（版本 2026.3.17）

这是 **飞书官方（Lark Suite）维护的版本**，不是社区版。

之前确实有两个飞书插件：
1. **社区版** - OpenClaw 社区维护的独立插件
2. **官方版** - 飞书官方维护的 `@larksuite/openclaw-lark`

现在扩展目录 `~/.openclaw/extensions/` 里只有 `openclaw-lark` 这一个，说明飞书官方版胜出了。官方版功能更全面，而且有飞书团队的持续维护支持。

所以你用的是**新版本 = 飞书官方版**。🎉
user: System: [2026-03-20 19:05:02 GMT+8] Feishu[default] DM | 黄威健 (ou_d7505506d0ca45a509cc90322e839302) [msg:om_x100b54e6d31160a4b3d6d538020608d]

Conversation info (untrusted metadata):
```json
{
  "message_id": "om_x100b54e6d31160a4b3d6d538020608d",
  "sender_id": "ou_d7505506d0ca45a509cc90322e839302",
  "sender": "黄威健",
  "timestamp": "Fri 2026-03-20 19:05 GMT+8"
}
```

Sender (untrusted metadata):
```json
{
  "label": "黄威健 (ou_d7505506d0ca45a509cc90322e839302)",
  "id": "ou_d7505506d0ca45a509cc90322e839302",
  "name": "黄威健"
}
```

你的子agent有单独运行在某个端口吗
assistant: 没有。所有 agent 共享同一个 Gateway：

- **Gateway 端口**：`ws://127.0.0.1:18789`（只有一个）
- **Agents**：3 个 agent（main、dev、shared）都通过这个端口

从 status 可以看到：
- `agent:main` 和 `agent:dev` 的 session 都在同一个 Gateway 下
- 它们只是逻辑隔离（不同的飞书账号、不同的工作空间），不是物理隔离

如果你想让 dev agent 单独跑一个端口，需要启动另一个 Gateway 实例（不同的配置目录）。但通常没必要——共享 Gateway 更省资源，agent 之间还能用 `sessions_send` 互相通信。
user: System: [2026-03-20 19:06:13 GMT+8] Feishu[default] DM | 黄威健 (ou_d7505506d0ca45a509cc90322e839302) [msg:om_x100b54e6ec87acb0b4b73be0e7ab2d0]

Conversation info (untrusted metadata):
```json
{
  "message_id": "om_x100b54e6ec87acb0b4b73be0e7ab2d0",
  "sender_id": "ou_d7505506d0ca45a509cc90322e839302",
  "sender": "黄威健",
  "timestamp": "Fri 2026-03-20 19:06 GMT+8"
}
```

Sender (untrusted metadata):
```json
{
  "label": "黄威健 (ou_d7505506d0ca45a509cc90322e839302)",
  "id": "ou_d7505506d0ca45a509cc90322e839302",
  "name": "黄威健"
}
```

你能告诉我配置子agent的具体流程吗
assistant: 好，我根据你的实际配置和官方文档整理一下：

---

## 配置子 Agent 的完整流程

### 1️⃣ 创建 Agent 工作空间

```bash
# 使用向导创建
openclaw agents add dev

# 或者手动创建目录
mkdir -p ~/.openclaw/workspace-dev
```

每个 agent 有独立的：
- **Workspace**：`~/.openclaw/workspace-dev`（放 AGENTS.md、SOUL.md、技能等）
- **AgentDir**：`~/.openclaw/agents/dev/agent`（放认证、模型配置）
- **Sessions**：`~/.openclaw/agents/dev/sessions`（会话历史）

---

### 2️⃣ 在 `openclaw.json` 中配置 Agent

```json5
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "name": "总指挥",
        "workspace": "/Users/weijian/.openclaw/workspace"
      },
      {
        "id": "dev",
        "name": "开发专家",
        "workspace": "/Users/weijian/.openclaw/workspace-dev",
        "skills": ["github", "gh-issues"],  // 可选：指定技能
        "model": "anthropic/claude-opus-4"   // 可选：覆盖默认模型
      }
    ]
  }
}
```

---

### 3️⃣ 创建飞书账号（每个 Agent 独立 Bot）

去 [飞书开放平台](https://open.feishu.cn/) 创建应用：
- 创建新应用 → 获取 `App ID` 和 `App Secret`
- 配置权限（消息、文档、日历等）
- 发布版本

---

### 4️⃣ 配置 Channel 账号

```json5
{
  "channels": {
    "feishu": {
      "enabled": true,
      "appId": "cli_a92fd8216c79dcd1",      // 主账号
      "appSecret": "Qhd1aKZyKgvNZAt0tUvP7znftKRDLv3G",
      "accounts": {
        "default": {},
        "dev": {
          "appId": "cli_a920c309463adcb1",   // dev 的飞书应用
          "appSecret": "QzyPQ1XsWqc75azF7hkmAdjYYzrsjxOd",
          "botName": "开发专家"
        }
      }
    }
  }
}
```

---

### 5️⃣ 配置 Bindings（路由规则）

```json5
{
  "bindings": [
    {
      "agentId": "main",
      "match": { "channel": "feishu", "accountId": "default" }
    },
    {
      "agentId": "dev",
      "match": { "channel": "feishu", "accountId": "dev" }
    }
  ]
}
```

**路由优先级**（从高到低）：
1. `peer` 匹配（精确到某个群/私聊）
2. `accountId` 匹配
3. 通道级别匹配
4. 默认 agent（`default: true`）

---

### 6️⃣ 重启生效

```bash
openclaw gateway restart
openclaw agents list --bindings
```

---

### 你当前的配置

| Agent | 飞书账号 | 工作空间 | 技能 |
|-------|---------|---------|------|
| main | default | ~/.openclaw/workspace | 全部 |
| dev | dev | ~/.openclaw/workspace-dev | github, gh-issues |
| shared | - | ~/.openclaw/workspace-shared | 只读 + 会话工具 |

两个飞书账号分别绑定到 main 和 dev，私聊它们会路由到不同的 agent。
