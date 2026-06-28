# study-with-claude-code · 二期蓝图（Phase-2 Blueprint）

> 状态:设计文档,未实现。一期(v0.3.0)已交付 `plan / go / stop / stats` 闭环。本文件规划剩余 5 个命令 + 闭环黏合 + 数据契约扩展 + 实现顺序。
> 原则承袭一期:每个命令 = 一个 `skills/<name>/SKILL.md`,按 compound-engineering 结构写(核心原则 / 执行流程表 / 门控 / 决策表 / 质量基准),复用 `_shared/` 契约与 rubric;质量靠流程(必要时 subagent)保证,不靠用户判断。
> 数据格式的**唯一真相源仍是 `_shared/data-contract.md`**——本文确定的契约扩展,实现时要落到那里。

---

## 0. 总览:还差什么

一期闭环能「建树 → 复习 → 评分 → 排下次」,但作为间隔复习工具有三类缺口:

| 类别 | 缺口 | 本蓝图对策 |
|---|---|---|
| **广度** | 9 件套还差 `switch / weak / mock / deep / compound` | 第 2–6 节逐个设计 |
| **黏合** | 到期复习**没有主动浮现入口**,艾宾浩斯算了却没人提醒;`dailyGoal` 定义了没人用 | 第 1 节「今日到期」黏合层 |
| **健壮** | 手动改树后与 progress 不对账;薄弱点「连续两次达标才移除」缺计数字段;`dir` 类资料未验证 | 第 7 节 |

实现顺序(依赖排序)见第 8 节。**最高杠杆的不是某个新命令,而是第 1 节的「今日到期浮现」**——它让工具真正被持续使用。

---

## 1. swcc-notice —— 「今天该复习什么」每日提醒（最高优先，独立 skill，cron 友好）

**问题**:`progress.json` 里每个节点/薄弱点都有 `nextReview`,但没有任何地方主动说「今天有 N 个该复习」。用户想不起来回来 → 间隔复习失效。

**对策**:做一个**专门的只读 skill `swcc-notice`**(而非塞进 stats),作为每日复习提醒入口。

- **触发**:「今天复习什么」「该复习啥了」「复习提醒」;以及**定时触发**(用户用 `/loop` 或 cron 每天唤起,变成每日提醒)。
- **参数**:`[topic]`(可选;默认**跨所有专题**汇总,这对每日提醒最有用)。
- **定义**:`due(today)` = 所有 `nextReview ≤ 今天(date +%F)` 的节点 + 薄弱点。
- **流程**:
  1. 读 `config.topics[]`,逐专题读 `progress.json`,收集 `due(today)`。
  2. 汇总:每个专题到期几个、哪些薄弱点最逾期(按 `nextReview` 升序)。
  3. 给**建议执行顺序**(逾期最久 / 薄弱点优先)并指向 `/swcc-weak`、`/swcc-go` 去做。
  4. 显示 `dailyGoal` 进度(今日已复习时长 / 目标分钟,从今天的 review-sessions 估)。
- **写**:**只读,零写入** —— 正因如此才适合无人值守的定时触发。
- **cron 友好**:无状态、只读、给定日期确定性输出。用户后续可 `CronCreate` 每天某个非整点时刻触发 `/swcc-notice`,把摘要当晨间提醒。
- **契约扩展**:无新结构,纯计算;需在 data-contract 写清 `due(today)` 定义与 `dailyGoal`(分钟)消费方式。

**输出草案**:
```
🔔 今天该复习（2026-06-29）
━━━━━━━━━━━━━━━━━━━━
redis：到期 3（薄弱点 2）
  · ZSet 跳表实现        逾期 2 天   → /swcc-weak redis
  · 缓存击穿与互斥重建    今天到期
jvm：到期 1
  · G1 Mixed GC 触发条件  逾期 5 天   → /swcc-weak jvm

今日目标 60min ｜ 已复习 0min
建议:先清 jvm 逾期 5 天那条，再过 redis 两个薄弱点。
```

> `stats` 仍可顺带显示当前专题的到期清单,但**主动浮现的家是 `swcc-notice`**。

---

## 2. switch —— 多专题切换 + 跨专题仪表盘（先做）

PRD 把 `list` 并入 `switch`:无参=列出+选,带参=直接切。

- **触发**:「切换到 X」「列出专题」「我有哪些专题」「switch」。
- **参数**:`[topic]`(可选)。
- **流程**:
  | 情况 | 行为 |
  |---|---|
  | 无参 | 读 `config.topics[]`,逐专题渲染:掌握比例 + **今日到期数** + 上次复习日;用 `AskUserQuestion` 让用户选 → 设 `activeTopic`。 |
  | 带参 | 匹配 slug → 设 `activeTopic` → 确认。匹配不到则列出可选项。 |
- **写**:仅 `config.json` 的 `activeTopic`(继 plan/stop 后第三个 writer)。
- **价值**:这是「今日到期」跨专题视图的家,也是多专题可用性的前提。
- **契约扩展**:无新结构;需读各专题 `progress.json` 算到期数(小心 IO,逐个读)。

**仪表盘输出草案**:
```
📚 我的复习专题
━━━━━━━━━━━━━━━━━━━━
▶ redis    🟢 12/25 掌握   🔔 今日到期 3   上次 06-27   ← 当前
  jvm      🟢  5/30 掌握   🔔 今日到期 0   上次 06-20
切换:/swcc-switch <topic>
```

---

## 3. weak —— 薄弱点专项强化（间隔复习的核心，第二做）

`weak` 是 `go` 的「只打薄弱点」变体,共用 `stop` 归档。

- **触发**:「复习薄弱点」「强化弱项」「weak」。
- **参数**:`[topic]`(默认 activeTopic)。
- **流程**:
  1. 读 `weakPoints[]`,按 `nextReview` 升序,**到期的优先**。
  2. 逐个薄弱点针对性强化:复用 `go` 的苏格拉底决策表,但**只问该薄弱概念相关**的问题,挖到「为什么/边界」。
  3. 只读、不写文件;收尾输出结构化小结(同 go),交给 `/swcc-stop`。
- **与 go/stop 的关系**:`weak` + `stop` 配对,和 `go` + `stop` 完全一样的归档路径。`stop` 已会更新 weakPoints(含达标移除)。
- **契约扩展(必须)**:薄弱点「**连续两次达标(mastery≥8)才移除**」目前缺计数字段。给 `weakPoints[]` 元素加 `consecutivePass`(达标 +1,未达标归 0,≥2 移除)。这条要先于 weak 落到 data-contract + 让 stop 维护它。
- **输出**:沿用 PRD 的薄弱点复习样式(逐条:上次/下次 + 问答 + 点评)。

---

## 4. mock —— 模拟面试（showcase，第三做）

跨节点连续追问 + 计时 + 综合评分报告。

- **触发**:「模拟面试」「面我一次」「mock」。
- **参数**:`[topic]` `[level]`(默认当前专题 / p7)。
- **题量与配比**(PRD):概念题 ×3、场景题 ×2、(可选)算法/实操题 ×1。**注意本插件是通用复习工具**,算法题只对编码类专题适用 → 由「专题类型」决定是否出(见契约扩展)。
- **流程**:
  1. 选题篮:**偏向薄弱点 + 覆盖主干**,按 level 调难度。
  2. 连续作答(可标注建议用时,不强制真计时)。
  3. 全部答完后,**派评估子智能体**按维度打分(承袭 stop 的客观评分思路,复用/扩展 mastery-rubric → 新建 `mock-rubric.md`),生成报告。
- **写**:新增 `topics/<slug>/mock-sessions/<时间戳>.md`(面试报告);并**回写 weakPoints**(答砸的题 → 薄弱点)、`stats`(计一次复习)。不直接改各节点 mastery(避免一次面试大改进度),只通过 weakPoints 影响后续。
- **契约扩展**:
  - 新目录 `mock-sessions/`。
  - 新文件 `_shared/mock-rubric.md`(各题型评分 + 总分合成 + 报告结构)。
  - **专题类型**:在 `config.topics[]` 或 knowledge-tree frontmatter 加 `kind: coding | conceptual`(决定是否出算法题、场景题口径)。默认 `conceptual`。
- **输出**:沿用 PRD 的「🎤 模拟面试报告」(总分 + 分题型得分 + 改进建议)。

---

## 5. deep —— 深挖拓展（生成型，独立，可随时插入）

把一个概念**横向拓展**(跨领域类比、技术对比、延伸问题),是「拓宽 + 沉淀」,不是测试。

- **触发**:「深挖 X」「拓展一下 GC」「deep」。
- **参数**:`[topic]` `[concept]`(concept 必填)。
- **流程**:
  1. 读知识树中该概念节点 + knowledge-system 内容(挂书则读相关章节)。
  2. 生成:跨领域连接 / 生活类比 / 相关技术对比 / 延伸开放问题(PRD 四段)。
  3. **沉淀**:写到 `topics/<slug>/deep-notes/<concept>.md`,让深挖结果累积成知识资产(契合「compound」愿景)。
- **写**:仅新增 `deep-notes/` 笔记,**不影响 mastery/progress**(纯增益)。
- **契约扩展**:新目录 `deep-notes/`。
- **subagent**:可选——用一个「广度」子智能体扩跨领域连接;但**不依赖联网**(默认靠模型知识 + 已挂资料),保证离线可用。

---

## 6. compound —— 阶段沉淀报告（趋势型，第四做）

与 `stats`(即时快照)区别:`compound` 是**跨时间趋势**报告 + 落盘产物 + 下阶段建议。

- **触发**:「复习报告」「阶段总结」「沉淀」「compound」。
- **参数**:`[topic]`(默认 activeTopic)。
- **流程**:
  1. 读 `review-sessions/`(+ `mock-sessions/`)全部记录,按日期聚合。
  2. 算:掌握度随时间变化、薄弱点新增/消除趋势、streak、总时长/总题数。
  3. 渲染报告并**写 `topics/<slug>/reports/<date>.md`**;给下阶段建议(进入哪个新专题 / 该加强什么 / 模拟面试频率)。
- **写**:新增 `reports/`。只读聚合,不改 progress。
- **契约扩展**:新目录 `reports/`;依赖 review-sessions 已有 date+mastery(满足)。
- **输出**:沿用 PRD 的「📈 复习报告」(总时长/次数/streak + 掌握度 ASCII 曲线 + 薄弱点趋势 + 下阶段建议)。

---

## 7. 健壮性 / 收尾

- **专题判定优先级(定死成原则)**:`go / weak / stop / stats` 解析专题时一律按 **显式参数 > 当前对话内容 > `activeTopic`(兜底)**。尤其 `stop` 必须从**对话里实际复习的专题**归档,不认 activeTopic——这样「忘了 switch」最坏只是裸跑默认命令时考错专题(go 开头会打印当前专题,当场可见可纠),**绝不会把成绩写错专题**。这条要落到 data-contract 并在各 SKILL 复述。
- **手动改树对账**:用户手动编辑 `knowledge-tree.md` 后,tree 与 `progress.json` 可能漂移。提供一个**对账步骤**(可做成 `plan --reconcile` 或 `stats` 启动时静默检测):按节点路径 key 比对,tree 有而 progress 无 → 补 `not_started`;progress 有而 tree 无 → 标孤儿提示。复用 data-contract 第十一节合并语义。
- **`dailyGoal` 落地**:stats/switch 显示「今日已复习 X / 目标 Y 分钟」。
- **`dir` 类参考资料**:`references.json` 的 `type: dir` 遍历策略已写但未验证,需在 plan/go 真实跑一次。
- **大文件分块写(通用,已在 v0.3.1 落地)**:`mock`/`compound` 的长报告、以及 plan 的 `knowledge-system.md`,都**禁止一次性整文件输出**(会超时),一律按 data-contract 第十二节「先骨架后分块 Edit 追加 + TodoWrite 跟踪 + 大产物可并行起草」来写。
- **subagent 成本观察**:plan/stop 的子智能体让 token/延迟上升,mock 又加一个。真实使用里观察是否过重,必要时给「轻量模式」开关(跳过评审)。

---

## 8. 实现顺序（依赖排序）

| 步骤 | 交付 | 依赖 / 新契约 |
|---|---|---|
| **0** | **先验证 v0.3.0**(redis 端到端) | 无——但必须先做,别在未验证地基上叠 |
| 1 | `swcc-notice`（每日提醒）+ `switch` | 无新结构;定义 `due(today)` + `dailyGoal` 消费 |
| 2 | `weak` | 给 weakPoints 加 `consecutivePass`,stop 维护它 |
| 3 | `mock` | `mock-sessions/`、`_shared/mock-rubric.md`、专题 `kind` |
| 4 | `compound` | `reports/`(聚合 sessions + mock) |
| 5 | `deep` | `deep-notes/`(独立,可提前/延后) |
| 6 | 健壮性收尾(对账 / dir 资料 / 成本观察) | —— |

> 每步都是一次独立的「改契约(如需)→ 写 SKILL.md → bump 版本 → push → 用户更新 → 验证」小循环,延续一期的发布工作流。

---

## 9. 命令全景（一期已交付 + 二期规划）

| 命令 | 状态 | 一句话 | 写哪 |
|---|---|---|---|
| `swcc-plan` | ✅ v0.3.0 | 建/更新 roadmap 级知识树 + 评审子智能体 | tree/system/progress/refs/config |
| `swcc-go` | ✅ v0.3.0 | 苏格拉底式逐层深挖复习 | 只读 |
| `swcc-stop` | ✅ v0.3.0 | 独立子智能体客观评分 + 归档 | review-session/progress/tree |
| `swcc-stats` | ✅ v0.3.0 | 即时进度快照（+ 今日到期，二期增强） | 只读 |
| `swcc-notice` | 🔵 二期 | 「今天该复习什么」每日提醒（cron 友好） | 只读 |
| `swcc-switch` | 🔵 二期 | 多专题切换 + 跨专题到期仪表盘 | config.activeTopic |
| `swcc-weak` | 🔵 二期 | 薄弱点专项强化（配 stop 归档） | 只读 |
| `swcc-mock` | 🔵 二期 | 模拟面试 + 评分报告 | mock-sessions/progress(weakPoints) |
| `swcc-compound` | 🔵 二期 | 阶段沉淀趋势报告 | reports/ |
| `swcc-deep` | 🔵 二期 | 概念横向深挖 + 沉淀笔记 | deep-notes/ |
