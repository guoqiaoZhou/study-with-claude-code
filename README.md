# study-with-claude-code

> AI 驱动的通用复习系统 —— 以 Claude Code 插件（一组 skill）的形式，帮你建立知识树、闭环复习、自动评估掌握度，并按艾宾浩斯遗忘曲线安排复习。不限于某个技术栈，任何技术专题都能用。

## 它解决什么

把"看完就忘"的零散学习，变成一个可追踪的闭环：

```
建知识树  →  逐题复习  →  系统评估掌握度 + 记薄弱点  →  到期自动提醒复习
  /plan         /go              /stop                       /stats
```

- **知识树自动生成**：给个专题（如 JVM），自动拆成结构化知识树 + 知识体系文档，可挂载 PDF/文档作为依据。
- **闭环复习**：`/go` 自动定位到当前该复习的点（优先到期薄弱点），逐题考核并点评。
- **系统评估掌握度**：`/stop` 由 AI 按「知识面覆盖度 + 回答深度」打分，不靠自评；自动记录薄弱点。
- **间隔复习**：按艾宾浩斯曲线 `[1,2,4,7,15]` 天安排下次复习。
- **数据是你的**：所有复习数据存在 `~/.in-prep/`，独立于插件，更新插件不会丢。

## 安装

```bash
# 添加本仓库为 marketplace
claude plugin marketplace add guoqiaoZhou/study-with-claude-code

# 安装插件
claude plugin install study-with-claude-code
```

本地开发/试用（不安装）：

```bash
claude --plugin-dir /path/to/study-with-claude-code
```

## 命令

| 命令 | 用法 | 作用 |
|------|------|------|
| `/study-with-claude-code:plan` | `plan [topic] [level]` | 生成专题知识树 + 知识体系，可挂载参考资料 |
| `/study-with-claude-code:go` | `go [topic] [mode]` | 开始复习，自动定位知识点逐题考核 |
| `/study-with-claude-code:stop` | `stop` | 结束并归档，系统评估掌握度、记薄弱点、排下次复习 |
| `/study-with-claude-code:stats` | `stats [topic]` | 查看进度：掌握比例、薄弱点、连续天数、统计 |

- `level`：`p6`/`p7`/`p8`/`p9`（深度，默认 `p7`）
- `mode`：`concept`/`scenario`/`mixed`（复习模式，默认 `mixed`）
- ⚠️ `go` 与 `stop` 需在**同一轮对话**里使用——`stop` 要读本次对话来归档。

## 典型流程

```
/plan JVM p7          # 建 JVM 知识树（会问你要不要挂《深入理解JVM》PDF）
/go                   # 开始复习 → 答题 → AI 点评
/stop                 # 归档，自动评分 + 记薄弱点 + 排下次复习
/stats                # 看进度
```

## 数据存储

```
~/.in-prep/
├── config.json                       # 全局配置 + 专题注册表
└── topics/<topic>/
    ├── knowledge-tree.md             # 知识树骨架（嵌套 checkbox）
    ├── knowledge-system.md           # 详细知识体系
    ├── progress.json                 # 进度 / 掌握度 / 薄弱点 / 统计
    ├── references.json               # 参考资料引用清单（只记路径）
    └── review-sessions/              # 每次复习的归档记录
```

参考资料**只记录原始路径，不复制文件**。PDF 用 Claude Code 原生分页读取（零依赖），docx 尽力转换、不行则提示转格式。

## 路线图

- **一期（当前）**：`plan` / `go` / `stop` / `stats` —— 核心复习闭环。
- **后续**：`weak`（薄弱点强化）、`mock`（模拟面试）、`deep`（跨领域深挖）、`compound`（阶段沉淀报告）、`switch`（多专题切换）；`plan` 增量迭代。

## License

MIT
