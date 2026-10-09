# 🔥 DingZhen Hot Monitor — 你自己的热点监控 Skill 使用指南

这份指南回答两件事：
1. **怎么把 `dingzhen-hot-monitor` 这个 skill 变成"你自己的"并装进各种 AI 工具**
2. **怎么把它跑起来、自己配置**

> 核心概念：Agent Skill 是一个「文件夹 + 一份 `SKILL.md`」，装进 Cursor / VSCode Copilot / Claude Code 等 AI 编程工具后，工具的 AI 就能直接调用里面的脚本来查热点、分析并生成报告。它**不需要数据库、不需要单独启动服务**，脚本抓数据、AI 做分析。

---

## 目录结构

`dingzhen-hot-monitor` 就是你自己的 skill，位于：

```
skills/dingzhen-hot-monitor/
├── SKILL.md                        # 技能定义（名称 + 描述 + 工作流 + 报告模板）
├── references/
│   ├── search-sources.md           # 各信息源的技术细节、频率限制
│   └── analysis-guide.md           # 真假/相关性/重要性分析框架
└── scripts/
    ├── search_web.py               # 国际源: Bing / Google / DuckDuckGo / HackerNews
    ├── search_china.py             # 中文源: 搜狗 / B站 / 微博
    ├── search_twitter.py           # 推特源（需要可选 key）
    ├── generate_report.py          # 把 JSON 转为 Markdown 报告
    └── requirements.txt            # Python 依赖清单
```

---

## 一、把 skill 装进 AI 工具

### 通用前提：Python 环境

skill 的脚本是 Python 写的，先在任意终端里装好依赖（只需一次）：

```bash
pip install requests beautifulsoup4
```

> 如果电脑提示没有 `pip`，先装 Python 3（≥3.8 即可），或在你的环境管理器里装。

### Claude Code（~/.claude/skills 全局 或 项目 .claude/skills）

```bash
# 全局安装(所有项目可用)
mkdir -p ~/.claude/skills
cp -r skills/dingzhen-hot-monitor ~/.claude/skills/

# 或 只装到当前项目
mkdir -p .claude/skills
cp -r skills/dingzhen-hot-monitor .claude/skills/
```

装好后在 Claude Code 里问「最近 AI 有什么热点」即可触发。

### Cursor（项目 .cursor/skills）

```bash
mkdir -p .cursor/skills
cp -r skills/dingzhen-hot-monitor .cursor/skills/
```

Cursor 会自动识别 `skills/` 目录下的 skill。重启 Cursor 后生效。

### VSCode / GitHub Copilot（自定义指令）

VSCode Copilot 使用 `.github/copilot-instructions`（项目级）或 `AGENTS.md` 来注入自定义指令。最好的做法是让 Copilot 知道这份 skill 的存在：

在项目根目录的 `AGENTS.md` 中追加一行指向你 skill 的说明，例如：

```markdown
## Hot Monitor Skill
项目自带热点监控技能 `skills/dingzhen-hot-monitor/`。当用户想查热点、跟踪关键词、生成热点报告时，请读取其中的 `SKILL.md` 并按其工作流执行（调用 scripts 下的 Python 脚本收集数据，再自行分析并生成报告）。
```

### DeepSeek Harness（当前 DSH 环境）

把整个 `dingzhen-hot-monitor` 文件夹放到 DSH 的 skills 目录即可被识别：

```
C:\Users\Lenovo\.dsh\skills\dingzhen-hot-monitor\SKILL.md
```

复制完成后需要重启 / 重新加载 session 才会出现在可用技能列表。

> 💡 **想要"更像你自己的"**：
> - 改 `SKILL.md` 开头的 `name:`（技能名)和 `description:`(触发描述，改成你习惯的说法)
> - 改报告模板里的语气/字段
> - 把仓库里的 README、版权署名改成你自己的信息
> - 你甚至可以把整个项目推到你自己名下的 GitHub(记得先改 LICENSE 和署名)

---

## 二、自己配置 skill

skill 只有**一个可选配置**：Twitter 的 key。其余数据源全部免 key、免费。

### 可选：配置 Twitter（国际热点更全）

| 环境变量 | 来源 | 是否必需 |
|----------|------|----------|
| `TWITTER_API_KEY` | [twitterapi.io](https://twitterapi.io/dashboard) | 否（不配则跳过推特） |

在运行脚本的终端里设置，例如：

```bash
# Windows (PowerShell)
$env:TWITTER_API_KEY="你的key"

# macOS / Linux (bash)
export TWITTER_API_KEY="你的key"
```

### Python 依赖清单

```bash
pip install requests beautifulsoup4   # 全部脚本依赖
```

---

## 三、手动跑起来（命令行验证）

安装好依赖后，可以在任何终端直接调用脚本，**不需要启动任何服务**。

### 1. 查国际源

```bash
cd skills/dingzhen-hot-monitor/scripts
python search_web.py "AI 编程" --sources bing,hackernews
```

### 2. 查中文源

```bash
python search_china.py "AI编程" --sources sogou,bilibili,weibo
```

### 3. 追踪某个 B 站 UP 主

```bash
python search_china.py "加格" --detect-account
```

### 4. 生成报告（把搜索输出管道给报告脚本）

```bash
python search_web.py "GPT-5" --sources hackernews | python generate_report.py
```

每个脚本都在 `stdout` 输出 JSON，失败时输出 `[]` 并在 `stderr` 打印原因。常见参数：`--sources`（指定数据源）、`--limit`（每源条数，默认 20）。

> 脚本要拿到正确搜索效果，请在 `scripts/` 目录下运行（避免相对路径问题）。

---

## 四、在 AI 工具里用（正经用法）

装好 skill 后，你不用手动敲脚本，直接对 AI 说就行，例如：

| 你想做的 | 直接对 AI 说 |
|----------|--------------|
| 泛泛了解热点 | 「最近 AI 领域有什么热点？」 |
| 跟踪特定关键词 | 「帮我关注 GPT-5 的最新动态」 |
| 追踪某个账号/UP主 | 「看看 加格 最近发了什么」 |
| 生成日报 | 「生成今天的 AI 热点报告」 |

skill 会自动调用脚本、多源汇总、用 AI 判断真伪/相关性/重要性，并按模板输出报告。

---

## 五、附：整个 Web 应用怎么跑（不想用就不看）

如果你还想跑那个**完整的前后端监控网站**（带界面、数据库、定时任务、邮件通知），那就不是 skill 的事了，需要启动 `server/` + `client/` 两个服务，并按下面的步骤：

### 1. 装依赖并建库

```bash
cd server
npm install
npx prisma generate
npx prisma db push

cd ../client
npm install
```

### 2. 配置 DeepSeek key（AI 分析用）

复制环境变量模板并填入你的 DeepSeek Key：

```bash
copy server\.env.example server\.env   # Windows
# 或: cp server/.env.example server/.env   # macOS/Linux
```

编辑 `server/.env`，至少填 `DEEPSEEK_API_KEY`：

```env
DATABASE_URL="file:./dev.db"
PORT=3001
CLIENT_URL=http://localhost:5173
DEEPSEEK_API_KEY=sk-你的key    # ← 在 platform.deepseek.com 创建
```

> Web 应用默认用 `deepseek-chat` 模型，想换深度推理用 `deepseek-reasoner`，可用 `DEEPSEEK_MODEL` 覆盖。

### 3. 启动两个终端

```bash
# 终端 1：后端 (端口 3001)
cd server && npm run dev

# 终端 2：前端 (端口 5173)
cd client && npm run dev
```

浏览器打开 **http://localhost:5173** 即可开始监控。