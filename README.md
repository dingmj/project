# 🔥 加格热点监控

> 作者：加格
>
> 一个用 AI 帮你盯热点的工具：输入监控关键词，自动从 8 个信息源聚合抓取，用 AI 识别真假、判断相关性和重要程度，再通过浏览器实时推送 + 邮件通知你。

---

## 一、这是什么

作为 AI 编程博主，我需要第一时间发现最新的热点（比如某个大模型又更新了），并且及时收到通知，好走在吃瓜第一线。

与其每天手动刷各个平台，不如做一个更通用的工具：**让 AI 帮你盯热点，第一时间获取优质信息。**

这个工具同时提供两种使用方式：

| 使用方式 | 适合场景 | 是否需要 API Key |
|----------|----------|------------------|
| **Web 应用**（前后端） | 想用界面管理关键词、看实时热点流、收通知 | 需要 DeepSeek API Key |
| **Agent Skill 技能包** | 在 Cursor / Claude Code / VSCode Copilot 里直接查热点 | 不需要（分析交给 AI 工具自身） |

---

## 二、核心功能

**1）关键词监控**

添加你想追踪的关键词，可以随时激活 / 暂停 / 编辑 / 删除。

**2）AI 多源聚合抓取与分析**

从 8 个信息源聚合抓取内容，然后交给 AI 做这些事：

- **查询扩展**：把关键词自动扩展成多个变体（中英对照、缩写、别称），提高召回率
- **真假识别**：排除标题党、假新闻、营销软文
- **相关性打分**：0-100 分，判断内容是否真的直接涉及你的关键词
- **重要性分级**：`urgent` / `high` / `medium` / `low`
- **一句话摘要**：说明这条内容和你关键词的关联

**3）多维度筛选和排序**

按来源、重要性、时间范围筛选；按热度、相关性、时间排序。

**4）全网搜索**

不依赖监控词，直接输入关键词从多个数据源聚合搜索。

**5）实时通知**

WebSocket 实时推送到浏览器，重要热点同时发邮件。

**6）定时自动巡检**

每 **30 分钟**自动跑一轮关键词巡检，发现新热点自动分析入库并推送。

**7）Agent Skill 技能包**

把热点监控能力封装成标准 Agent Skill，装进 AI 编程工具即可使用（详见第六节）。

---

## 三、技术栈

| 层级 | 技术 |
|------|------|
| 前端 | React 19 + Vite + TailwindCSS 4 + Framer Motion + Lucide Icons |
| 后端 | Node.js + Express 5 |
| 数据库 | SQLite + Prisma |
| AI 服务 | DeepSeek 官方 API（OpenAI 兼容） |
| 实时通信 | Socket.io |
| 定时任务 | node-cron |
| 邮件 | Nodemailer |

---

## 四、数据源

共 **8 个**信息源，除 Twitter 外全部免费、无需 Key：

| 来源 | 方式 | 需要 Key |
|------|------|----------|
| Bing | 网页抓取 | ❌ |
| Google | 网页抓取 | ❌ |
| DuckDuckGo | 网页抓取 | ❌ |
| HackerNews | 官方 Algolia API | ❌ |
| 搜狗 | 网页抓取 | ❌ |
| B站 | 公开 API | ❌ |
| 微博热搜 | 公开 API | ❌ |
| Twitter / X | twitterapi.io | ✅ 可选 |

抓取时会做 **URL 去重**、**7 天新鲜度过滤**，并按来源优先级排序（Twitter > 微博 > B站 > HackerNews > 搜索引擎）。

---

## 五、快速开始（Web 应用）

### 前置条件

- Node.js ≥ 18（推荐 20 LTS）
- 一个 [DeepSeek API Key](https://platform.deepseek.com/api_keys)（必需，用于 AI 分析）

### 1. 安装依赖

```bash
# 后端
cd server
npm install
npx prisma generate
npx prisma db push

# 前端
cd ../client
npm install
```

### 2. 配置环境变量

```bash
cp server/.env.example server/.env
```

编辑 `server/.env`，至少填入 DeepSeek API Key：

```env
DATABASE_URL="file:./dev.db"
PORT=3001
CLIENT_URL=http://localhost:5173

# 必填：DeepSeek AI
DEEPSEEK_API_KEY=sk-你的key

# 选填：Twitter 数据源
TWITTER_API_KEY=你的key

# 选填：邮件通知
SMTP_HOST=smtp.qq.com
SMTP_PORT=465
SMTP_SECURE=true
SMTP_USER=你的邮箱@qq.com
SMTP_PASS=你的授权码
NOTIFY_EMAIL=接收通知的邮箱@example.com
```

> 💡 可选配置：`DEEPSEEK_MODEL`（默认 `deepseek-chat`，可换 `deepseek-reasoner`）、`DEEPSEEK_BASE_URL`（默认 `https://api.deepseek.com`）。

### 3. 启动服务（两个终端）

```bash
# 终端 1：后端（端口 3001）
cd server && npm run dev

# 终端 2：前端（端口 5173）
cd client && npm run dev
```

访问 **http://localhost:5173**，输入关键词即可开始监控 🔥

| 服务 | 地址 |
|------|------|
| 前端页面 | http://localhost:5173 |
| 后端 API | http://localhost:3001 |
| 数据库管理 | `cd server && npx prisma studio`（可选） |

> 更详细的保姆级教程见 [本地运行指南](docs/LOCAL_SETUP.md)。

---

## 六、Agent Skill 技能包

热点监控能力已封装为 Agent Skill，位置：

```
skills/dingzhen-hot-monitor/
├── SKILL.md
├── references/          # 信息源细节 + 分析框架
└── scripts/             # 搜索与报告脚本（Python）
```

**特点**：不需要数据库、不需要启动任何服务、**不需要 API Key**——脚本负责抓数据，分析交给 AI 工具自身完成。

### 安装到 AI 工具

```bash
# Claude Code
cp -r skills/dingzhen-hot-monitor ~/.claude/skills/

# Cursor
mkdir -p .cursor/skills && cp -r skills/dingzhen-hot-monitor .cursor/skills/
```

脚本依赖：

```bash
pip install requests beautifulsoup4
```

### 直接命令行使用

```bash
cd skills/dingzhen-hot-monitor/scripts

# 国际源
python search_web.py "AI programming" --sources bing,hackernews

# 中文源
python search_china.py "AI编程" --sources sogou,bilibili

# 生成报告
python search_web.py "GPT-5" --sources hackernews | python generate_report.py --keyword "GPT-5"
```

安装后在 AI 工具里直接说「最近 AI 有什么热点」「帮我关注 GPT-5 动态」即可。

> 详见 [Skill 使用指南](docs/SKILL_GUIDE.md)。

---

## 七、项目结构

```
dingzhen-hot-monitor/
├── docs/                          # 文档
│   ├── LOCAL_SETUP.md            # 本地运行指南
│   ├── SKILL_GUIDE.md            # Skill 使用指南
│   ├── API_INTEGRATION.md        # API 集成细节
│   ├── REQUIREMENTS.md           # 需求说明
│   └── README.md                 # 项目概述
├── server/                        # 后端
│   ├── src/
│   │   ├── services/
│   │   │   ├── ai.ts             # DeepSeek 分析 + 查询扩展
│   │   │   ├── search.ts         # 国际源搜索
│   │   │   ├── chinaSearch.ts    # 中文源搜索
│   │   │   ├── twitter.ts        # Twitter 搜索
│   │   │   └── email.ts          # 邮件通知
│   │   ├── routes/               # keywords / hotspots / notifications / settings
│   │   ├── jobs/hotspotChecker.ts # 定时巡检任务
│   │   ├── db.ts                 # Prisma 客户端
│   │   └── index.ts              # 服务入口
│   └── prisma/schema.prisma
├── client/                        # 前端
│   └── src/
│       ├── App.tsx               # 主界面（热点雷达 / 监控词 / 搜索）
│       ├── components/           # UI 组件
│       └── services/             # API + WebSocket 客户端
└── skills/dingzhen-hot-monitor/   # Agent Skill
```

---

## 八、主要 API

| 方法 | 路径 | 说明 |
|------|------|------|
| GET / POST | `/api/keywords` | 获取 / 新增监控关键词 |
| PUT / DELETE | `/api/keywords/:id` | 编辑 / 删除关键词 |
| PATCH | `/api/keywords/:id/toggle` | 激活 / 暂停关键词 |
| GET | `/api/hotspots` | 热点列表（支持筛选排序） |
| GET | `/api/hotspots/stats` | 统计数据 |
| POST | `/api/hotspots/search` | 手动触发全网搜索 |
| GET / PATCH | `/api/notifications` | 通知列表 / 标记已读 |
| GET / PUT | `/api/settings` | 设置读写 |

---

## 九、常见问题

**Q：热点搜索没有结果？**
1. 检查 `server/.env` 中 `DEEPSEEK_API_KEY` 是否填写、余额是否充足
2. 关键词太冷门 → 换成更热的关键词试试（如 `AI`、`ChatGPT`）
3. 某些搜索引擎反爬可能导致部分源失败 → 看后端日志，其他源仍会正常返回

**Q：Twitter 没有数据？**
未配置 `TWITTER_API_KEY`（可选），不影响其他 7 个源。

**Q：不填 DeepSeek Key 能跑吗？**
能启动、能抓取，但 AI 分析会降级为默认分数（不判断真假、相关性固定、重要性都是 `low`）。想看完整效果必须配 Key。

**Q：数据库文件在哪？**
`server/prisma/dev.db`，用 `npx prisma studio` 可视化查看。

---

## 十、端口汇总

| 服务 | 默认端口 |
|------|----------|
| 后端 API | 3001 |
| 前端页面 | 5173 |
| Prisma Studio | 5555（可选） |
