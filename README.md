# study-with-claude-code

> AI 驱动的通用复习系统 —— 以 Claude Code 插件（一组 skill）的形式，帮你建立知识树、闭环复习、系统评估掌握度、按艾宾浩斯曲线安排复习，并让系统**越学越懂你**。不限技术栈，任何专题都能用。

## 它解决什么

把"看完就忘"的零散学习，变成一个会**复利**的闭环：

```
建知识树 → 每天看该复习啥 → 先讲后测的闭环复习 → 系统评估+记薄弱点 → 复盘让系统更懂你
 /swcc-plan     /swcc-daily        /swcc-go            /swcc-stop          /swcc-compound
```

- **知识树自动生成**：给个专题，自动拆成结构化知识树 + 知识体系，可挂 PDF/文档为依据；**生成后强制用评审子智能体查漏补缺**。可对已有专题增量更新、**已学进度不丢**。
- **先讲后测、从第一性讲起**：`/swcc-go` 先带你把知识点过一遍——复杂原理**从本质/设计哲学讲起**（先给对的视角再讲机制），讲完给你多个深挖方向；**再苏格拉底式 + 费曼检验**考核。薄弱点**只从考核阶段记**，不会把"还没复习"误记成弱项。
- **系统客观评估掌握度**：`/swcc-stop` 派**独立子智能体**按「覆盖度 + 深度」打分，不靠自评；按艾宾浩斯 `[1,2,4,7,15]` 天排下次复习。
- **越学越懂你（复利）**：`/swcc-compound` 和你聊"学得怎么样、哪里吃力、为什么"，提炼出关于**你怎么学**的档案（讲解偏好、反复盲区、有效引导），让 go/weak/mock 之后把你教得越来越贴——只调风格、不松纪律。
- **沉淀成果**：`/swcc-blog` 把本轮学习/讨论写成可读 blog（默认存 `./blogs/`，可发布）；`/swcc-deep` 把概念横向深挖。
- **数据是你的**：复习数据存 `~/.study-with-cc/`，独立于插件，更新不丢。

## 安装

```bash
claude plugin marketplace add guoqiaoZhou/study-with-claude-code
claude plugin install study-with-claude-code
```

本地试用（不安装）：`claude --plugin-dir /path/to/study-with-claude-code`

## 命令（12 个）

> 不知道用哪个?直接 **`/swcc-help`** —— 它会按"我想做 X → 用哪个命令"帮你选。

**核心闭环**

| 命令 | 用法 | 作用 |
|------|------|------|
| `/swcc-plan` | `[topic] [level] [--update]` | 生成/更新 roadmap 级知识树 + 知识体系（评审子智能体查漏补缺） |
| `/swcc-go` | `[topic] [study\|test] [concept\|scenario\|mixed]` | 先讲后测：从第一性讲透 → 苏格拉底考核 |
| `/swcc-stop` | `stop` | 结束并归档：独立子智能体评分、记薄弱点、排下次复习 |
| `/swcc-stats` | `[topic]` | 进度快照：掌握比例、薄弱点、连续天数、统计 |

**日常 & 多专题**

| 命令 | 用法 | 作用 |
|------|------|------|
| `/swcc-daily` | `[topic]` | 今天该复习什么（跨专题到期汇总，只读，可定时提醒） |
| `/swcc-switch` | `[topic]` | 多专题切换 + 总览 |
| `/swcc-weak` | `[topic]` | 薄弱点专项强化（配 `/swcc-stop` 归档） |

**进阶 & 沉淀**

| 命令 | 用法 | 作用 |
|------|------|------|
| `/swcc-mock` | `[topic] [level]` | 模拟面试 + 独立评分报告 |
| `/swcc-compound` | `[topic]` | 学习复盘：提炼「你怎么学」让系统更懂你 |
| `/swcc-deep` | `[topic] [concept]` | 概念横向深挖（可选联网） |
| `/swcc-blog` | `[拆分提示] [out:目录]` | 把本轮学习/讨论沉淀成 blog |

- `level`：`p6`/`p7`/`p8`/`p9`（深度，默认 `p7`）
- ⚠️ `go`/`weak` 与 `stop` 需在**同一轮对话**里使用——`stop` 要读本次对话来归档。

## 典型流程

```
/swcc-plan <主题> p7       # 建知识树（会问要不要挂 PDF/资料）
/swcc-daily               # 每天看今天该复习啥
/swcc-go → 答题 → /swcc-stop   # 先讲后测 → 归档评分
/swcc-weak → /swcc-stop        # 专攻薄弱点
/swcc-compound            # 阶段复盘，让系统更懂你
/swcc-blog 3              # 把学到的写成 3 篇 blog
```

## 数据存储

```
~/.study-with-cc/
├── config.json            # 全局配置 + 专题注册表
├── learner-profile.md     # 学习者档案（跨专题：你怎么学；compound 维护，各技能读取调风格）
└── topics/<topic>/
    ├── knowledge-tree.md / knowledge-system.md / progress.json / references.json
    ├── review-sessions/   # go/weak 归档    mock-sessions/  # 模拟面试报告
    ├── reports/           # compound 复盘   deep-notes/     # 深挖笔记
```

blog 默认写到**当前工作目录** `./blogs/`（便于发布），不在插件数据目录里。参考资料只记路径不复制；PDF 用原生分页读取（零依赖）。

## License

MIT
