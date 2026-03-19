# 媒体分析技能设计说明书

> 本文档描述如何创建一个**媒体分析报告生成技能**，包括数据源配置、执行流程、子Agent协作方式等。Agent应根据此说明书生成实际的技能实现。

---

## 一、技能概述

### 1.1 功能目标

用户输入一个话题，技能自动：
1. 搜索多个媒体平台获取相关信息
2. 整合数据生成结构化分析报告
3. 输出完整的 Markdown 格式报告

### 1.2 触发场景

- 用户说"分析一下XXX话题"
- 用户说"帮我做个话题调研"
- 用户说"生成XXX的媒体报告"

---

## 二、数据源配置

### 2.1 优先级排序

| 优先级 | 数据源 | 访问方式 | 需要配置 |
|--------|--------|---------|---------|
| **1** | Tavily API | HTTP API | API Key |
| **2** | 今日头条 | HTTP → 解析 JSON | 无 |
| **3** | 微信公众号 | HTTP → 解析 HTML | 无 |
| **4** | 微博移动端 API | HTTP + Cookie | 需获取 Cookie |
| **5** | Bing CN | HTTP → 解析 HTML | 无 |

### 2.2 各数据源详情

#### Tavily API（首选）

**优势**：一次请求获取多个来源的结果，结构化 JSON 返回

**接口**：
```bash
POST https://api.tavily.com/search
Authorization: Bearer <TAVILY_API_KEY>
Content-Type: application/json

{
  "query": "<话题>",
  "max_results": 10,
  "search_depth": "advanced"
}
```

**配置**：在 `~/.claude/settings.json` 中设置：
```json
{
  "env": {
    "TAVILY_API_KEY": "tvly-xxx"
  }
}
```

#### 今日头条

**优势**：新闻资讯丰富，时效性强，JSON 嵌入在页面中

**接口**：
```bash
GET https://so.toutiao.com/search?keyword=<话题>
```

**解析**：用正则提取 `"title":"xxx"` 等字段

#### 微信公众号

**优势**：深度分析文章多，观点质量高

**接口**：
```bash
GET https://wx.sogou.com/weixin?type=2&query=<话题>
```

**说明**：type=2 搜索文章，type=1 搜索公众号

#### 微博移动端

**优势**：社交讨论、热度指标、实时性强

**接口**：
```bash
GET https://m.weibo.cn/api/container/getIndex?containerid=100103type=1&q=<话题>
```

**注意**：需要 Cookie（可通过浏览器获取）

#### Bing CN

**优势**：知乎内容多，结构稳定

**接口**：
```bash
GET https://cn.bing.com/search?q=<话题>
```

---

## 三、执行流程

### 3.1 主流程

```
用户输入话题
    ↓
[Step 1] 调用 Tavily search/research
    ↓
结果是否足够？ ─否─→ [Step 2] 启动子Agent补充搜索
    ↓ 是
[Step 3] 整合数据
    ↓
[Step 4] 生成结构化报告
    ↓
输出 Markdown 文档
```

### 3.2 子Agent协作流程

当 Tavily 结果不足时，启动多个子Agent并行搜索：

```
                    ┌→ 子Agent 1: 今日头条搜索
                    │
主Agent → 启动子Agent ─┼→ 子Agent 2: 微信公众号搜索
                    │
                    ├→ 子Agent 3: 微博搜索
                    │
                    └→ 子Agent 4: Bing CN搜索
                              ↓
                    主Agent 汇总结果生成报告
```

### 3.3 子Agent任务模板

**今日头条搜索 Agent**：
```
任务：搜索今日头条获取话题相关新闻
URL: https://so.toutiao.com/search?keyword=<话题>
提取：
- 新闻标题
- 发布时间
- 内容摘要
- 热度指标（播放量等）
返回：JSON 格式结果
```

**微信公众号搜索 Agent**：
```
任务：搜索微信公众号文章
URL: https://wx.sogou.com/weixin?type=2&query=<话题>
提取：
- 文章标题
- 公众号名称
- 发布时间
- 文章摘要
返回：JSON 格式结果
```

**微博搜索 Agent**：
```
任务：搜索微博获取社交讨论
URL: https://m.weibo.cn/api/container/getIndex?containerid=100103type=1&q=<话题>
需要：Cookie（从浏览器获取）
提取：
- 微博内容
- 用户名
- 互动数据（转发/评论/点赞）
返回：JSON 格式结果
```

---

## 四、报告结构

生成的报告应包含以下章节：

```markdown
# 📰 XXX话题媒体分析报告

> 报告生成时间：YYYY-MM-DD | 数据来源：XXX

## 📌 事件概述
- 事件名称、时间、地点
- 核心当事人
- 事件性质

## ⏱️ 完整时间线
按时间顺序列出关键事件

## 📊 各平台热度分析
| 平台 | 热度 | 主要内容 |
|------|------|---------|

## 💬 舆论焦点
- 主要讨论话题
- 不同观点阵营
- 热门评论摘要

## 📰 媒体报道分析
- 主流媒体报道倾向
- 自媒体观点分布

## 🎯 结论与洞察
- 事件本质判断
- 社会影响评估
- 后续发展预测
```

---

## 五、实现方式

### 5.1 作为技能实现

在 `~/.openclaw/workspace/.agents/skills/media-analyze/` 目录创建：

```
media-analyze/
├── SKILL.md          # 技能描述文档
├── scripts/
│   └── analyze.sh    # 执行脚本
└── README.md         # 使用说明
```

### 5.2 作为 Agent 能力实现

在 Agent 的系统提示中添加：

```
当用户要求分析某个话题时：

1. 首先调用 Tavily search API 搜索
2. 如果结果不足，启动子Agent补充搜索：
   - 今日头条
   - 微信公众号
   - 微博
3. 整合数据，生成结构化报告
```

### 5.3 纯脚本实现

创建一个独立的 shell 脚本或 Python 脚本，按流程执行各平台搜索并整合输出。

---

## 六、注意事项

### 6.1 数据时效性
- 报告中标注数据获取时间
- 热点话题可能快速变化

### 6.2 来源标注
- 每个数据点标注来源URL
- 方便用户验证

### 6.3 观点平衡
- 呈现多方观点
- 避免单一立场

### 6.4 隐私保护
- 对敏感个人信息脱敏
- 不传播隐私数据

### 6.5 错误处理
- API 限制时降级使用其他数据源
- 平台访问被拒时跳过或使用备用方案
- 数据不足时明确告知用户

---

## 七、快速开始

### 安装步骤

1. **配置 Tavily API Key**
```bash
mkdir -p ~/.claude
echo '{"env":{"TAVILY_API_KEY":"tvly-xxx"}}' > ~/.claude/settings.json
```

2. **创建技能目录**
```bash
mkdir -p ~/.openclaw/workspace/.agents/skills/media-analyze
```

3. **测试执行**
```
对 Agent 说："帮我分析一下 XXX 话题的媒体报道"
```

### 示例调用

```
用户: 帮我分析一下武汉大学图书馆事件的媒体报道

Agent 执行:
1. 调用 Tavily search "武汉大学图书馆事件"
2. 整合搜索结果
3. 生成结构化报告
4. 输出 Markdown 文档
```

---

## 八、附录

### 8.1 Tavily API 完整参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| query | string | 必填 | 搜索关键词 |
| max_results | int | 10 | 最大结果数(1-20) |
| search_depth | string | basic | basic/advanced |
| time_range | string | null | day/week/month/year |
| include_domains | array | [] | 限定域名 |

### 8.2 今日头条 JSON 提取

```python
import re
# 提取标题
titles = re.findall(r'"title":"([^"]+)"', html)
# 提取播放量
play_counts = re.findall(r'"play_count":"([^"]+)"', html)
```

### 8.3 微信公众号 HTML 解析

```python
from bs4 import BeautifulSoup
soup = BeautifulSoup(html)
articles = soup.select('.news-list li')
for article in articles:
    title = article.select_one('h3').text
    source = article.select_one('.s2').text
    date = article.select_one('.s3').text
```

---

## 九、版本历史

| 版本 | 日期 | 说明 |
|------|------|------|
| v1.0 | 2026-03-16 | 初始设计文档 |

---

**文档结束**。Agent 应根据此说明书实现媒体分析技能的具体功能。