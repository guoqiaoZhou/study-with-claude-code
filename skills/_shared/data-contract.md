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
    ├── review-sessions/
    │   └── <YYYY-MM-DD-HH-mm-ss>.md     # go/weak 每次复习一份归档记录
    ├── mock-sessions/                   # mock 每次模拟面试一份报告（按需创建）
    │   └── <YYYY-MM-DD-HH-mm-ss>.md
    ├── reports/                         # compound 阶段沉淀报告（按需创建）
    │   └── <YYYY-MM-DD>.md
    └── deep-notes/                      # deep 概念深挖笔记（按需创建）
        └── <concept-slug>.md
```

> 操作任何文件前，若目录不存在需先 `mkdir -p`。所有 JSON 用 2 空格缩进、UTF-8、保留中文原文（不转义 unicode）。

---

## 二、节点标识：层级路径文本

知识树是多层结构，progress.json 需要引用具体节点。**统一用节点的「层级路径文本」作为唯一 key**，从知识树的二级标题/列表项拼接，用 `/` 连接，**不含最顶层 topic 名**。

示例：知识树里 `## <子主题>` 下有 `- [ ] <中间节点>`，其下有 `- [ ] <叶子节点>`，则该叶子节点的 key 为：

```
<子主题>/<中间节点>/<叶子节点>
```

各层名称用 `/` 连接，不含最顶层 topic 名。
- 人类可读，progress.json 和 knowledge-tree.md 之间靠这个路径互相对应。
- 不引入隐藏 id（一期简化）。代价是重命名节点会断链——一期可接受。
- go / stop 读取时按路径做精确匹配。

---

## 三、knowledge-tree.md（结构骨架）

YAML frontmatter + Markdown 标题 + 嵌套 `- [ ]` checkbox。**只存结构与状态，不存详细内容**（详细内容在 knowledge-system.md）。

```markdown
---
topic: <主题名>
slug: <主题-slug>
version: 1.0
level: p7
kind: conceptual
createdAt: 2026-06-28
---

# <主题名>

## <子主题 A>
- [ ] <知识点 A1>
- [ ] <知识点 A2>
  - [ ] <更细的子知识点>

## <子主题 B>
- [ ] <知识点 B1>
  - [ ] <子知识点 B1a> 🔴
```

**frontmatter 字段**:`topic`/`slug`/`version`/`level`/`createdAt` 见示例;`kind` = `coding`(编程类,可出算法/实操题) 或 `conceptual`(概念类,**默认**),由 plan 生成时按专题性质判定,供 mock 决定是否出算法/实操题。

**checkbox 状态标记**：
- `- [ ]` 未掌握 / 学习中
- `- [x]` 已掌握（mastery ≥ 8 时由 stop 勾选）
- 行尾加 `🔴` 表示该节点是薄弱点（与 progress.json 的 weakPoints 对应）

**节点粒度（重要）**:tree 是骨架，深度内容在 knowledge-system.md。骨架本身也要**足够细**——每个叶子节点应是一个「可单独考核的知识点」，而不是含义过宽、无法单独出题的大词。若一个节点名太笼统，就把它拆成几个能各自出题的下级节点（例如按「底层实现与代价」「适用场景与取舍」等不同角度拆分）。`##` 二级标题作分组层，其下可再嵌一层。核心问题、原理、来源等深度内容**不写进 tree**，只写进 knowledge-system.md；tree 只保留结构与勾选状态。

---

## 四、knowledge-system.md（系统化总结 / 学习锚点，roadmap 级）

**定位（重要）**:它是这个专题的**系统化总结 + 学习地图**，**不是逐字教材**。职责是把"该掌握什么"结构化下来——每个叶子节点的**核心问题、核心概念要点（浓缩）、面试考点、以及深度内容的来源指引（哪章书 / 网络补充）**。它保证**覆盖面与结构**，是 go / weak / mock 的**索引与出题依据**。

**它不负责"逐字讲透"**:真正的讲解深度在 go 的 study 阶段**临场生成**——以本文为大纲，结合**挂载的资料（读对应章节原文）+ 模型自身深度知识**，把每个要点展开成 blog 级。所以本文每个节点写到"系统化要点"即可（带答案、带原理方向、带来源指引），**不必也不应把每个点扩成长篇**（否则文件巨大且生成超时）。密度按 review-rubric「深度标准」那五条衡量。

> 一句话:**system 是地图，不是领土**。go 负责带用户走进领土;读 system 只能拿到地图,讲解深度必须超过 system 本身。

每个叶子节点四段:

```markdown
# <主题名> 知识体系

## <子主题> / <叶子节点>

**核心问题**：<一句驱动性问题，定调本节点要回答什么>

### 核心概念
- <要点一：先给结论，再讲背后的原理/机制，再说与替代方案的取舍，最后点出边界或易错点>
- <要点二：……>（覆盖该节点全部关键概念，每条都达到「结论 + 原理 + 取舍 + 边界」）

### 面试考察点
1. <该 level 真实会被问到的具体问题>
2. <……>

### 来源
- 书：<第 N 章> ｜ 网络补充：<资料未覆盖、但该 level 该掌握的点>
```

要点:
- **核心问题**:一句驱动性问题，定调本节点要回答什么，逼出深度。
- **核心概念**:多条实质要点，每条自带答案与原理/取舍/边界，**不要只写名词**。
- **面试考察点**:该 level 真实会问的具体题。
- **来源**:标清哪部分来自挂载的资料（第几章）、哪部分是 `网络补充`。**没挂资料就只写自身理解，不要假称来源**（见第七、八节）。

---

## 五、progress.json（进度 / 掌握度 / 薄弱点 / 统计）

```json
{
  "project": "<主题-slug>",
  "currentNode": "<子主题>/<叶子节点>",
  "nodes": {
    "<子主题>/<叶子节点>": {
      "status": "not_started",
      "mastery": 0,
      "lastReviewed": null,
      "nextReview": null,
      "reviewCount": 0
    }
  },
  "weakPoints": [
    {
      "node": "<子主题>/<另一个节点>",
      "concept": "<考核时暴露的具体薄弱概念>",
      "mastery": 4,
      "createdAt": "2026-06-28",
      "nextReview": "2026-06-29",
      "reviewCount": 0,
      "consecutivePass": 0
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
- `currentNode`：当前/下一个该复习的节点路径，go 用它定位。**推进规则**:节点本次 `mastery ≥ 8` → 推进到下一个 `not_started` 叶子(先同子主题、再下个子主题);`mastery < 8` → 保持该节点;用户明确要求换下一个 → 推进但该节点保持 `in_progress`。
- `weakPoints`：考核阶段卡壳/低掌握的具体点，由 stop 写入。**两个独立计数,别混用**:`reviewCount` = 该薄弱点被复习过几次(用于艾宾浩斯间隔);`consecutivePass` = 连续达标次数(考核 `mastery ≥ 8` 则 +1、否则归 0)。**`consecutivePass ≥ 2` 时移除该薄弱点。**
- `stats.streak`：**全局**(跨所有专题)连续复习天数,不是按专题。`lastReviewDate` = 昨天 → +1;= 今天 → 不变(今天已复习过);否则 → 重置为 1。
- 日期一律 `YYYY-MM-DD`；时间戳 `YYYY-MM-DD-HH-mm-ss`。`totalReviewTime` 单位为分钟。

---

## 六、艾宾浩斯复习间隔

默认间隔数组 `[1, 2, 4, 7, 15]`（天），取自 config 的 `reviewSettings.weakReviewInterval`。

计算 `nextReview`：以本次复习日期为基准，取间隔数组第 `min(reviewCount, len-1)` 个（reviewCount 从 1 开始计）。例：第 1 次复习后 +1 天，第 2 次 +2 天，第 3 次 +4 天……第 5 次及以后 +15 天。

**用谁的 reviewCount**:
- **节点**的 `nextReview` → 用**该节点**本次复习后的 `reviewCount`。
- **薄弱点**的 `nextReview` → 用**该薄弱点自身**的 `reviewCount`。
- 二者独立计数,不要互相套用。

> 当前日期从 shell 取：`date +%F`（本地时区）。skill 内不要臆造日期。

---

## 七、references.json（参考资料引用清单，topic 级）

**只记录原始路径，不复制文件内容**（方案 A，省空间，原文件留在原处）。

```json
{
  "references": [
    {
      "type": "pdf",
      "path": "/Users/<user>/books/<某教材>.pdf",
      "title": "<教材标题>",
      "note": "<作者 / 版次等备注>",
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
  "activeTopic": "<主题-slug>",
  "topics": [
    { "name": "<主题-slug>", "path": "topics/<主题-slug>", "status": "active", "level": "p7", "createdAt": "2026-06-28" }
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

**兜底（重要）**：任何技能若发现 `config.json` 不存在或缺 `activeTopic`，应**自愈**而不是报错退出：
1. 扫描 `$HOME/.study-with-cc/topics/` 下的子目录，得到已有专题列表（每个子目录名即 slug）。
2. 只有一个专题 → 直接用它，并就地补建/补全 `config.json`（activeTopic 指向它、topics[] 补上该条）。
3. 有多个 → 列出来让用户指定要用哪个，然后补全 config.json。
4. 一个都没有 → 提示用户先 `/swcc-plan <topic>`。

---

## 十、复习模式（mode）

go 的 `mode` 含两层:

**取向（学还是考）**:
- `study`（**默认**）— **先讲后测**:先带用户把节点核心概念过一遍(讲透、点出易忽略细节、可下钻),再考核。适合生疏/遗忘/首次。
- `test` — **直接考**:跳过讲解直接苏格拉底式考核。适合已熟、快速检验,或定位到的是已学过的到期薄弱点。

**考核题型（只作用于考核阶段）**:
- `concept` — 概念复习:逐个考核节点核心概念
- `scenario` — 场景题:结合实际场景应用
- `mixed`（默认）— 概念 + 场景混合

**关键:掌握度/薄弱点只看考核阶段。** 讲解阶段(study 取向的"先过一遍")不评判、不记薄弱点——否则会把"还没复习"误记成薄弱点。只有"讲过/复习过、考核仍卡"才算薄弱点。详见 mastery-rubric 第四节。

> 纯 `weak`（薄弱点专项，天然 test 取向）作为独立 skill 见二期蓝图;`algo` 题型留待后续。

---

## 十一、合并 / 更新语义（plan 更新模式用）

`plan` 对**已存在**的专题再次运行时，默认走**增量合并**（不是覆盖、不是停止），核心是**保住用户已有进度**。统一以**节点层级路径文本**（第二节）为 key 做对齐:

| 情况 | 处理 |
|---|---|
| 新大纲里**新增**的节点（旧 tree 没有） | tree 加该行；progress.json 的 `nodes` 补一条 `not_started` / `mastery:0`。 |
| **两边都有**的节点（路径相同） | knowledge-system.md 内容**可加深/修订**；progress.json 里该节点的 status/mastery/lastReviewed/nextReview/reviewCount **一律不动**。 |
| 旧 tree 有、但新大纲**不再包含**的节点 | 视为**进度孤儿**:tree 与 progress 里**保留不删**，并在摘要里提示用户（一期不自动删，避免误伤已学进度）。 |

其它:
- weakPoints、stats 在合并模式下**原样保留**，只随新增节点自然增长。
- knowledge-tree.md frontmatter 的 `version` 递增（如 `1.0` → `1.1`）。
- `--regenerate`（全量重建）:重写 tree 与 system，但仍按路径 key **迁移旧 progress**——能匹配上的节点把旧 status/mastery 搬过来，匹配不上的旧节点在摘要里列出由用户决定。**默认不要用 regenerate**，除非用户明确要求。

> 原则:**结构可演进，进度不可丢**。任何更新都以「用户已学的不白学」为底线。

---

## 十二、大文件分块写入策略（避免超时，所有技能通用）

**为什么要分块**:`knowledge-system.md` 往往有几十个节点、每个都要展开成深度内容，**一次性整文件输出会因单条响应过长而超时失败**。任何可能很大的产物（knowledge-system.md、未来的 mock 报告、compound 报告）都**禁止一次性写完**，必须分块。

**标准协议（分块追加）**:

1. **先写骨架**:用 Write 创建文件，只放表头 + 一个末尾哨兵行，例如:
   ```markdown
   # <topic> 知识体系

   <!-- @end -->
   ```
2. **建 TodoWrite 清单**:为每个 `##` 子主题建一个 todo（「生成+落盘 <子主题>」），逐个推进、实时勾掉，让进度可见、可续。
3. **逐块追加**:每次只处理**一个子主题**，把该子主题下所有叶子节点的内容用 **Edit 替换哨兵**追加:
   ```
   Edit  old="<!-- @end -->"  new="<本子主题的完整内容>\n\n<!-- @end -->"
   ```
   一个子主题 = 一次 Edit = 一次有界输出。**绝不**把多个子主题攒在一条里。
4. **收尾**:全部写完后可把末尾 `<!-- @end -->` 删掉（可选）。
5. **大专题可并行起草**:子主题很多时，用 Agent 工具**并行派起草子智能体**，每个只负责**一个子主题**、**只返回该子主题的 markdown 文本、不写文件**（避免并发写同一文件冲突）；主体拿到各段后仍按第 3 步**逐块 Edit 追加**（追加本身是串行、有界的）。

**适用面**:本协议对**任何大文件写入**通用——`progress.json` 节点很多时同理可分批构造；`compound`/`mock` 的长报告也按此分块。判断标准:**预计输出会很长，就先骨架后分块，配 TodoWrite 跟踪。**

---

## 十三、到期判定 due(today) 与 dailyGoal（daily / switch / stats 共用）

**`due(today)`**:某专题里所有满足 `nextReview ≤ 今天(date +%F)` 的**节点** + **薄弱点**。`nextReview` 为 `null`（从未复习）的节点**不算到期**——它属于"未开始"，不是"该复习了"。逾期天数 = 今天 − `nextReview`（≥0）。

**跨专题汇总**:遍历 `config.topics[]`，逐个读其 `progress.json` 收集 `due(today)`。排序按 `nextReview` 升序（逾期最久的在前）。

**`dailyGoal`**（`config.reviewSettings.dailyGoal`，单位分钟）:今日已复习时长 = 今天日期的所有 `review-sessions/` / `mock-sessions/` 记录里 `totalReviewTime` 估值之和（或从 `stats` 近似）。展示为「已复习 X / 目标 Y 分钟」。数据不足就只显示目标、不编已复习值。

> 这三处都是**只读计算**，不写任何文件。
