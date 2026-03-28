# 独立子 Agent 配置最佳实践

> 配置时间：2026-03-28
> 配置示例：helper agent

---

## 🎯 核心原则

### 1. **Agent 不改系统配置**
- ❌ Agent 不要承诺"我会修改 openclaw.json"
- ✅ Agent 生成配置片段 → 用户手动添加 → Agent 验证

### 2. **无端口配置**
- ❌ 不要为每个 agent 分配端口
- ✅ 所有 agent 共用一个 Gateway（单进程架构）

### 3. **三要素独立**
- ✅ **workspace**：独立工作目录（隔离文件）
- ✅ **飞书账号**：独立 bot（隔离身份）
- ✅ **技能白名单**：按需配置（隔离权限）

---

## 📋 配置流程（标准五步）

### Step 1：创建 Workspace
```bash
mkdir -p ~/.openclaw/workspace-{agent-id}
```

创建基础文件：
- `AGENTS.md` - 工作空间说明
- `SOUL.md` - Agent 人设
- `USER.md` - 用户信息（可继承）

### Step 2：生成配置片段
Agent 生成 JSON 片段，**不直接写入系统文件**：

```json
// agents.list 配置
{
  "id": "{agent-id}",
  "name": "{agent名称}",
  "workspace": "/Users/weijian/.openclaw/workspace-{agent-id}",
  "model": {
    "primary": "zai/glm-4.7"  // 默认模型
  },
  "subagents": {
    "allowAgents": ["main"]  // 或 allowAny: true
  }
}
```

### Step 3：用户手动添加
用户打开 `~/.openclaw/openclaw.json`，在三个位置添加：

1. `agents.list` → agent 定义
2. `channels.feishu.accounts` → 飞书账号
3. `bindings` → 绑定关系

### Step 4：重启 Gateway
```bash
openclaw gateway restart
```

### Step 5：验证
- 飞书搜索 bot 名称 → 发消息测试
- 主 agent 调用 `sessions_send` → 协作测试

---

## ⚠️ 常见错误

### 错误 1：Agent 改错配置文件
**现象**：Agent 说"配置完成"，实际改的是 workspace 里的模拟文件
**后果**：Gateway 无法识别新 agent，配置无效
**解决**：只生成片段，让用户手动操作

### 错误 2：配置端口
**现象**：配置里有 `"port": 18791`
**后果**：无效配置，单 Gateway 架构不支持
**解决**：删除端口配置项

### 错误 3：accountId 不一致
**现象**：bindings 里写 `"accountId": "helper"`，accounts 里没这个 key
**后果**：绑定失败，agent 无法启动
**解决**：三个位置的 ID 要完全一致

---

## 🔧 示例配置（helper agent）

### agents.list
```json
{
  "id": "helper",
  "name": "Helper助手",
  "workspace": "/Users/weijian/.openclaw/workspace-helper",
  "model": {
    "primary": "zai/glm-4.7"
  },
  "subagents": {
    "allowAgents": ["main"]
  }
}
```

### channels.feishu.accounts
```json
"helper": {
  "appId": "cli_xxx",
  "appSecret": "xxx",
  "botName": "Helper助手"
}
```

### bindings
```json
{
  "agentId": "helper",
  "match": {
    "channel": "feishu",
    "accountId": "helper"
  }
}
```

---

## 💡 调度机制

### Agent 间通信
主 agent 通过 `sessions_send` 调用子 agent：

```
sessions_send(
  sessionKey: "agent:{子agent-id}:...",
  message: "任务内容"
)
```

### 白名单配置
```json
"tools": {
  "agentToAgent": {
    "enabled": true,
    "allow": ["main", "dev", "helper"]
  }
}
```

---

## 📝 总结

**配置独立子 agent 的关键**：
1. ✅ Workspace 目录 + 基础文件（AGENTS/SOUL/USER）
2. ✅ 生成配置片段 → 用户手动添加
3. ✅ 无端口，单 Gateway
4. ✅ accountId 三处一致
5. ✅ 重启 Gateway → 验证

**贾维斯的教训**：
- 不要让 agent 自动改系统配置
- 系统文件（openclaw.json）是用户的手动操作区
- Agent 只负责：生成内容、验证结果

---

*Created: 2026-03-28*
*Author: 大白 + 星期五（协作总结）*