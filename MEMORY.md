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

## 待办事项

| 优先级 | 事项 | 状态 |
|--------|------|------|
| 高 | SOW 真实会议纪要测试 | ⏳ |
| 中 | SOW README 文档 | ⏳ |
| 中 | 知识库目录创建 | ⏳ 待确认 |
| 低 | 多模板扩展 | 未来迭代 |

---

*最后更新：2026-03-19*