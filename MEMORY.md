# 核心记忆

## 用户信息
- **姓名**：黄威健
- **飞书 ID**：`ou_d7505506d0ca45a509cc90322e839302`
- **邮箱**：huangweijian@cmcm.com
- **背景**：即将大学毕业，主要做AI应用开发
- **技术栈**：Python、JavaScript（脚本类开发），学过Java
- **方向**：考虑转型 Go + AI 应用开发

---

## Agent 配置

### 当前架构（精简后）
| Agent | 用途 | 状态 |
|-------|------|------|
| `main` | 总指挥（默认） | ✅ 运行中 |
| `dev` | 开发专家 | ✅ 独立飞书账号 |

### 开发专家 (dev) 配置
- **飞书账号**：独立 bot（开发专家）
- **App ID**：`cli_a920c309463adcb1`
- **技能**：github, gh-issues
- **Codex 调用**：直接用 `codex exec`（不用 ACP，会崩溃）
- **工作空间**：`~/.openclaw/workspace-dev`

### 模型配置
- **默认模型**：`bailian/glm-5`
- **代理端口**：7890（必须设置才能访问 OpenAI）

---

## 如何创建独立 Agent

### 架构说明
独立 Agent 支持**两种使用方式**：
1. **直接对话**：用户在飞书找该 Agent 的 bot 直接聊天
2. **协作调度**：其他 Agent 可以通过 `sessions_send` 调用它

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

### 步骤一：创建飞书应用（独立 Bot）

1. 访问 [飞书开放平台](https://open.feishu.cn/app)
2. 创建企业自建应用
3. 记录 `App ID` 和 `App Secret`
4. 配置权限（根据 agent 用途选择）：
   - 基础：`contact:user.base:readonly`
   - 消息：`im:message`, `im:message:send_as_bot`
   - 文档：`docx:document`, `drive:drive` 等
5. 发布版本 → 审核通过 → 添加到企业

### 步骤二：修改 `~/.openclaw/openclaw.json`

#### 2.1 在 `agents.list` 添加新 Agent

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

#### 2.2 配置飞书账号（在 `channels.feishu.accounts`）

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

#### 2.3 绑定 Agent 到飞书账号（在 `bindings`）

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

#### 2.4 允许 Agent 间通信（在 `tools.agentToAgent`）

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

### 步骤三：创建工作空间

```bash
mkdir -p ~/.openclaw/workspace-新agent-id
cd ~/.openclaw/workspace-新agent-id

# 可选：复制模板文件
cp ~/.openclaw/workspace/AGENTS.md .
cp ~/.openclaw/workspace/SOUL.md .
cp ~/.openclaw/workspace/USER.md .
```

### 步骤四：重启 Gateway

```bash
openclaw gateway restart
```

### 配置示例（完整）

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

### 验证

1. **直接对话**：在飞书搜索 bot 名称，发消息测试
2. **协作调度**：在 main 中调用 `sessions_send`：
   ```
   sessions_send(
     sessionKey: "agent:dev:feishu:dev:direct:ou_xxx",
     message: "测试消息"
   )
   ```

### 注意事项

- **accountId 要一致**：`bindings[].match.accountId` = `channels.feishu.accounts` 的 key
- **工作空间独立**：每个 agent 有自己的 workspace，互不干扰
- **技能可定制**：不同 agent 可以装不同的技能
- **权限控制**：用 `allowAny` 或 `allowAgents` 控制谁能调谁

---

## 已安装技能

### 自建技能
| 技能 | 用途 | 状态 |
|------|------|------|
| **sow-generator** | 项目工作说明书生成 | ✅ GitHub: hwj123hwj/sow-generator |
| **media-analyze** | 媒体分析报告生成 | 🚧 设计中 |

### 第三方技能
| 技能 | 用途 | 状态 |
|------|------|------|
| **DVCode 生图** | 香蕉Pro AI生图 | ✅ API Key 已登录 |
| **B站技能** | B站视频解析/字幕提取 | ✅ yt-dlp 已安装 |

### 飞书技能（16个）
从 GitHub `hwj123hwj/openclaw-skills` 安装：
- feishu-bitable, feishu-calendar, feishu-task
- feishu-doc-create, feishu-wiki-write
- feishu-send-message, feishu-read-message
- feishu-permission-setup, feishu-app-rename
- 等...

---

## 重要项目记录

### 1. SOW 生成器
**仓库**：https://github.com/hwj123hwj/sow-generator

**设计亮点**：
- 占位符系统：`{{变量名}}` 标记需填充内容
- 三层填充策略：直接填充 / 推荐填充 / 默认填充
- 复用度高：数据安全、变更管理、特别说明章节几乎完全复用

**待完成**：
- README 文档
- 真实会议纪要测试

### 2. 向量记忆架构决策
**场景**：500篇 × 4000字 ≈ 200万字

**结论**：OpenClaw 内置向量库完全够用（千级文档舒适区）

**推荐目录结构**：
```
~/.openclaw/workspace/
├── MEMORY.md           # 长期记忆
├── memory/             # 会话记录
└── knowledge-base/     # 用户知识库（独立管理）
```

---

## 技术问题与解决方案

### 1. acpx + Codex ACP 崩溃
- **问题**：退出码 5
- **解决**：直接用 `codex exec`，不用 `sessions_spawn(runtime="acp")`

### 2. 飞书权限批量取消无效
- **问题**：JavaScript `click()` 无法触发 React 合成事件
- **解决**：必须用 Playwright 真实点击

### 3. 代理配置
- **端口**：7890
- **环境变量**：HTTP_PROXY, HTTPS_PROXY, ALL_PROXY
- **配置文件**：`~/.zshrc`
- **用途**：访问 OpenAI/ChatGPT/Codex API 必须设置

---

## Go 语言学习（用户关注）

### 推荐项目
**云原生**：Kubernetes, Docker, Etcd, Consul
**网络服务**：Caddy, Traefik, Gin
**AI领域**：Ollama, LangChain Go, Milvus
**工具链**：Hugo, GoReleaser, Delve

### 学习建议
- Go + AI 互补：Python做AI，Go做工程
- 从 Gin、Hugo 入手，逐步深入分布式系统

---

## 其他配置

### Go SDK 环境
- **版本**：go1.26.1 darwin/arm64
- **安装路径**：`/Users/weijian/Desktop/develop/go`
- **配置文件**：`~/.zshrc`（GOROOT + PATH）

### DVCode API Key
- **Key**：`sk_live_Y21jbS5jb20_OwmLSXoDBDk6NnXbuhpt5u6TE7CSZHQ9`
- **有效期**：15天（自动续期）
- **注册地址**：https://dvcode.deepvlab.ai

---

## 毕设项目（VibeCheck）

### 项目信息
- **题目**：基于语义理解与情感分析的华语流行歌曲推荐系统设计与实现
- **指导老师**：陈茜
- **服务器**：49.233.41.129（腾讯云）

### 技术栈
- **后端**：Python + PostgreSQL + pgvector
- **前端**：Vue3 + Vite
- **特征体系**：TF-IDF（理性）+ LLM embedding（感性）
- **氛围标签**：治愈、伤感、热血、浪漫等

### 当前状态
- ✅ 系统已部署上线
- ✅ 5万+ 华语歌曲数据库
- ✅ 氛围搜索、歌词语义检索、智能推荐
- ⏳ 准备中期答辩（PPT需补充算法细节）
- ⏳ 用户调研、论文撰写

---

## 待办事项

| 优先级 | 事项 | 状态 |
|--------|------|------|
| 高 | 中期答辩准备 | 🔄 进行中 |
| 高 | SOW 真实会议纪要测试 | ⏳ |
| 中 | SOW README 文档 | ⏳ |
| 中 | 知识库目录创建 | ⏳ 待确认 |
| 低 | 多模板扩展 | 未来迭代 |

---

## 记忆仓库
- **GitHub**: https://github.com/hwj123hwj/dabai-memory
- **用途**: 大白专属记忆备份仓库
- **最后同步**: 2026-03-23

## 龙虾军团仓库
- **GitHub**: https://github.com/hwj123hwj/lobster-legion
- **用途**: 三主 agent（大白、星期五、贾维斯）协作管理，部署在不同物理服务器
- **结构**: 每主 agent 独立目录 + shared-knowledge 共享区

## boot-md Hook 经验
- BOOT.md 里写**指令**，不要写 `$(command)` shell 语法
- 内容要简单，复杂指令会导致 boot session 出错
- 配置: `openclaw hooks enable boot-md`，在工作区创建 BOOT.md

## Agent 架构更新（2026-03-28）
新增 agent:
| Agent | 用途 | 模型 |
|-------|------|------|
| `intel` | 情报助手 | zai/glm-4.7 |
| `money-assistant` | 赚钱助手 | zai/glm-4.7 |

---

*最后更新：2026-03-29*