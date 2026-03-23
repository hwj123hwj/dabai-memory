# 创建独立 Agent 完整指南

> 经验沉淀：2026-03-23
> 
> 场景：需要创建一个独立 Agent，支持用户直接对话 + 其他 Agent 协作调度

---

## 架构说明

独立 Agent 支持**两种使用方式**：

1. **直接对话**：用户在飞书找该 Agent 的 bot 直接聊天
2. **协作调度**：其他 Agent 通过 `sessions_send` 调用它

```
┌─────────────────────────────────────────────┐
│                    用户                       │
│         飞书私聊：找 main 或 找其他 agent       │
└─────────────────────────────────────────────┘
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
   ┌─────────┐◄──sessions_send──►┌──────────┐
   │  main   │     (协作通信)     │ 新 agent │
   └─────────┘                   └──────────┘
   独立飞书bot                    独立飞书bot
```

---

## 步骤一：创建飞书应用（独立 Bot）

1. 访问 [飞书开放平台](https://open.feishu.cn/app)
2. 创建企业自建应用
3. 记录 `App ID` 和 `App Secret`
4. 配置权限（根据 agent 用途选择）：
   - 基础：`contact:user.base:readonly`
   - 消息：`im:message`, `im:message:send_as_bot`
   - 文档：`docx:document`, `drive:drive` 等
5. 发布版本 → 审核通过 → 添加到企业

---

## 步骤二：修改 `~/.openclaw/openclaw.json`

### 2.1 在 `agents.list` 添加新 Agent

```json
{
  "id": "新agent-id",
  "name": "Agent名称",
  "workspace": "/Users/weijian/.openclaw/workspace-新agent-id",
  "skills": ["skill1", "skill2"],  // 可选：预装技能
  "subagents": {
    // 方式1：允许调用所有 agent
    "allowAny": true

    // 方式2：白名单（只能调指定的）
    // "allowAgents": ["main", "shared"]
  }
}
```

### 2.2 配置飞书账号（在 `channels.feishu.accounts`）

```json
{
  "channels": {
    "feishu": {
      "accounts": {
        "default": {},
        "dev": {
          "appId": "cli_xxx",
          "appSecret": "xxx",
          "botName": "开发专家"
        },
        "新agent-accountId": {
          "appId": "cli_yyy",
          "appSecret": "yyy",
          "botName": "新Agent显示名"
        }
      }
    }
  }
}
```

### 2.3 绑定 Agent 到飞书账号（在 `bindings`）

```json
{
  "bindings": [
    {
      "agentId": "main",
      "match": {
        "channel": "feishu",
        "accountId": "default"
      }
    },
    {
      "agentId": "新agent-id",
      "match": {
        "channel": "feishu",
        "accountId": "新agent-accountId"
      }
    }
  ]
}
```

### 2.4 允许 Agent 间通信（在 `tools.agentToAgent`）

```json
{
  "tools": {
    "agentToAgent": {
      "enabled": true,
      "allow": ["main", "dev", "新agent-id"]  // 添加到白名单
    }
  }
}
```

---

## 步骤三：创建工作空间

```bash
mkdir -p ~/.openclaw/workspace-新agent-id
cd ~/.openclaw/workspace-新agent-id

# 可选：复制模板文件
cp ~/.openclaw/workspace/AGENTS.md .
cp ~/.openclaw/workspace/SOUL.md .
cp ~/.openclaw/workspace/USER.md .
```

---

## 步骤四：重启 Gateway

```bash
openclaw gateway restart
```

---

## 配置示例（完整）

以 `dev` agent 为例：

```json
// agents.list
{
  "id": "dev",
  "name": "开发专家",
  "workspace": "/Users/weijian/.openclaw/workspace-dev",
  "skills": ["github", "gh-issues"],
  "subagents": {
    "allowAny": true
  }
}

// channels.feishu.accounts
"dev": {
  "appId": "cli_a920c309463adcb1",
  "appSecret": "QzyPQ1XsWqc75azF7hkmAdjYYzrsjxOd",
  "botName": "开发专家"
}

// bindings
{
  "agentId": "dev",
  "match": {
    "channel": "feishu",
    "accountId": "dev"
  }
}

// tools.agentToAgent.allow
["main", "dev", "shared"]
```

---

## 验证

### 1. 直接对话测试

在飞书搜索 bot 名称，发消息测试

### 2. 协作调度测试

在 main 中调用 `sessions_send`：

```javascript
sessions_send(
  sessionKey: "agent:dev:feishu:dev:direct:ou_xxx",
  message: "测试消息"
)
```

---

## 注意事项

1. **accountId 要一致**：`bindings[].match.accountId` = `channels.feishu.accounts` 的 key
2. **工作空间独立**：每个 agent 有自己的 workspace，互不干扰
3. **技能可定制**：不同 agent 可以装不同的技能
4. **权限控制**：用 `allowAny` 或 `allowAgents` 控制谁能调谁

---

## 关键配置位置总结

| 配置位置 | 作用 | 示例 |
|---------|------|------|
| `agents.list[]` | 定义 agent 本身 | `{ "id": "dev", ... }` |
| `channels.feishu.accounts` | 飞书 bot 账号 | `{ "appId": "cli_xxx", ... }` |
| `bindings` | 绑定 agent 到飞书账号 | `{ "agentId": "dev", "match": {...} }` |
| `tools.agentToAgent.allow` | agent 间通信白名单 | `["main", "dev"]` |
| `agents.list[].subagents` | agent 能调谁 | `{ "allowAny": true }` |

---

*经验来源：2026-03-23 实战配置*