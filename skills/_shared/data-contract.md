# 数据契约（Data Contract）

> 本文件是 study-with-claude-code 插件所有技能（plan / go / stop / stats）共享的**数据格式单一真相源**。每个 SKILL.md 在操作文件前都应先读它。任何对数据格式的修改只改这里。

---

## 一、存储位置

所有用户的复习数据存在**全局目录** `~/.study-with-cc/`（在 shell 里展开为 `$HOME/.study-with-cc`）。

**为什么放全局**：插件本体安装在 `~/.claude/plugins/cache/` 下，插件更新时该目录会被覆盖。用户数据绝不能放插件目录内，否则一更新就丢。`~/.study-with-cc/` 独立于插件，持久保存。

```
~/.study-with-cc/
├── config.json                          # 全局配置 + 专题注册表
└── topics/<topic>/                      # 每个专题一个目录，<topic> 为小写 kebab-case
    ├── knowledge-tree.md                # 结构骨架：YAML frontmatter + 嵌套 checkbox
    ├── knowledge-system.md              # 详细内容：每节点的概念解释 + 面试问题
    ├── progress.json                    # 进度 / 掌握度 / 薄弱点 / 统计
    ├── references.json                  # 参考资料引用清单（只记路径，不复制文件）
    └── review-sessions/
        └── <YYYY-MM-DD-HH-mm-ss>.md     # 每次复习一份归档记录
```

> 操作任何文件前，若目录不存在需先 `mkdir -p`。所有 JSON 用 2 空格缩进、UTF-8、保留中文原文（不转义 unicode）。

---

## 二、节点标识：层级路径文本

知识树是多层结构，progress.json 需要引用具体节点。**统一用节点的「层级路径文本」作为唯一 key**，从知识树的二级标题/列表项拼接，用 `/` 连接，**不含最顶层 topic 名**。

示例：知识树里 `## 垃圾回收` → `- [ ] G1` → `- [ ] Region 设计`，该叶子节点的 key 为：

```
垃圾回收/G1/Region 设计
```

- 人类可读，progress.json 和 knowledge-tree.md 之间靠这个路径互相对应。
- 不引入隐藏 id（一期简化）。代价是重命名节点会断链——一期可接受。
- go / stop 读取时按路径做精确匹配。

---

## 三、knowledge-tree.md（结构骨架）

YAML frontmatter + Markdown 标题 + 嵌套 `- [ ]` checkbox。**只存结构与状态，不存详细内容**（详细内容在 knowledge-system.md）。

```markdown
---
topic: JVM
slug: jvm
version: 1.0
level: p7
createdAt: 2026-06-28
---

# JVM

## 运行时数据区
- [ ] 程序计数器
- [ ] 虚拟机栈
- [ ] 堆
  - [ ] 新生代
  - [ ] 老年代

## 垃圾回收
- [ ] G1
  - [ ] Region 设计
  - [ ] Mixed GC 🔴
```

**checkbox 状态标记**：
- `- [ ]` 未掌握 / 学习中
- `- [x]` 已掌握（mastery ≥ 8 时由 stop 勾选）
- 行尾加 `🔴` 表示该节点是薄弱点（与 progress.json 的 weakPoints 对应）

---

## 四、knowledge-system.md（详细内容）

人类可读的详细文档，是 go 复习时的**出题与讲解依据**。按知识树结构组织，每个叶子节点给出：核心概念解释、面试考察点。沿用如下结构（可按专题调整）：

```markdown
# JVM 知识体系

## 垃圾回收 / G1 / Region 设计

### 核心概念
- 将堆划分为 1M-32M 的 Region……

### 面试考察点
1. G1 的 Region 大小怎么定？为什么是 2 的幂次？
2. ……
```

---

## 五、progress.json（进度 / 掌握度 / 薄弱点 / 统计）

```json
{
  "project": "jvm",
  "currentNode": "垃圾回收/G1/Region 设计",
  "nodes": {
    "垃圾回收/G1/Region 设计": {
      "status": "not_started",
      "mastery": 0,
      "lastReviewed": null,
      "nextReview": null,
      "reviewCount": 0
    }
  },
  "weakPoints": [
    {
      "node": "垃圾回收/G1/Mixed GC",
      "concept": "Mixed GC 触发条件",
      "mastery": 4,
      "createdAt": "2026-06-28",
      "nextReview": "2026-06-29",
      "reviewCount": 0
    }
  ],
  "stats": {
    "totalReviewTime": 0,
    "totalReviewCount": 0,
    "streak": 0,
    "lastReviewDate": null
  }
}
```

**字段说明**：
- `nodes` 的 key = 节点层级路径（见第二节）。plan 初始化时为每个**叶子节点**建一条，全部 `not_started` / `mastery: 0`。
- `status`：`not_started` → `in_progress` → `mastered`（mastery ≥ 8 时转 mastered）。
- `mastery`：0–10，**完全由系统评估**（见 stop 的评分规则），不是用户自评。
- `currentNode`：当前/下一个该复习的节点路径，go 用它定位。
- `weakPoints`：卡壳/低掌握的具体点，由 stop 写入；`mastery ≥ 8` 且连续两次达标后移除。
- 日期一律 `YYYY-MM-DD`；时间戳 `YYYY-MM-DD-HH-mm-ss`。`totalReviewTime` 单位为分钟。

---

## 六、艾宾浩斯复习间隔

默认间隔数组 `[1, 2, 4, 7, 15]`（天），取自 config 的 `reviewSettings.weakReviewInterval`。

计算 `nextReview`：以本次复习日期为基准，按该节点（或薄弱点）**复习后的 reviewCount** 取间隔数组第 `min(reviewCount, len-1)` 个（reviewCount 从 1 开始计）。例：第 1 次复习后 +1 天，第 2 次 +2 天，第 3 次 +4 天……第 5 次及以后 +15 天。

> 当前日期从 shell 取：`date +%F`（本地时区）。skill 内不要臆造日期。

---

## 七、references.json（参考资料引用清单，topic 级）

**只记录原始路径，不复制文件内容**（方案 A，省空间，原文件留在原处）。

```json
{
  "references": [
    {
      "type": "pdf",
      "path": "/Users/x/books/深入理解Java虚拟机.pdf",
      "title": "深入理解Java虚拟机",
      "note": "周志明，第3版",
      "addedAt": "2026-06-28"
    }
  ]
}
```

- `type`：`pdf` | `docx` | `txt` | `md` | `dir`（资料目录）。
- `path`：**绝对路径**。`dir` 表示一个目录，plan/go 按需遍历其中文件。
- 没有参考资料时，文件内容为 `{ "references": [] }`。

---

## 八、参考资料读取策略（plan / go 共用）

目标：**结合资料原文**生成知识树与复习内容，而非凭空发挥；同时**不把整本书塞进上下文**（省 token），按需读。

- **PDF** — 用 Read 工具**原生分页读**（`pages` 参数，每次最多 20 页，零外部依赖）。
  - `plan`：先读**目录页 / 前 10 页**提取章节结构，作为知识树分解依据。
  - `go`：只读与**当前复习节点**相关的章节页。
- **docx / doc** — Read 读不了二进制，做 **best-effort 转换**：
  1. 试 `pandoc <file> -t markdown`；
  2. 或 `markitdown <file>`；
  3. 都不可用 → 提示用户「请把该 docx 转成 PDF / txt / md 后重新挂载」，不报错退出。
- **txt / md** — 直接 Read。
- **dir** — 先列目录结构，再按与 topic / 当前节点的相关性挑文件，按上述规则读。

**原则**：插件不绑死任何外部依赖。PDF 走原生 Read，其它格式尽力而为，失败给清晰提示——保证在任何人的机器上都能跑起来。

---

## 九、config.json（全局配置 + 专题注册表）

```json
{
  "version": "1.0",
  "activeTopic": "jvm",
  "topics": [
    { "name": "jvm", "path": "topics/jvm", "status": "active", "level": "p7", "createdAt": "2026-06-28" }
  ],
  "reviewSettings": {
    "dailyGoal": 60,
    "weakReviewInterval": [1, 2, 4, 7, 15],
    "defaultMode": "mixed"
  }
}
```

- `activeTopic`：当前默认专题，go/stop/stats 不带 topic 参数时用它。
- `topics[]`：所有已建专题。plan 新建专题时追加一条并把 activeTopic 指向它。
- 首次运行（文件不存在）时由 plan 创建该文件。

---

## 十、复习模式（mode）

go 的 `mode` 参数，一期支持：
- `concept` — 概念复习：逐个考核节点核心概念
- `scenario` — 场景题：结合实际场景应用
- `mixed`（默认）— 概念 + 场景混合

> `algo` / 纯 `weak` 模式留待后续阶段。
