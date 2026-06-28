---
name: swcc-stop
description: Use when the user wants to end the current review session — must be the same conversation as the go/weak session it closes. 触发:「结束复习」「stop」「归档」「复习完了」「今天就到这」「收工」「收尾」「存一下进度」「记一下今天的」「先到这」「done」「完事了」。
user-invocable: true
---

# swcc · stop — 结束复习并归档

归档**当前这轮对话里**刚发生的复习:**独立评分**、记薄弱点、更新进度、排下次复习。

> 动手前先读两份契约:
> - `${CLAUDE_PLUGIN_ROOT}/skills/_shared/data-contract.md` —— 落盘格式与艾宾浩斯计算。
> - `${CLAUDE_PLUGIN_ROOT}/skills/_shared/mastery-rubric.md` —— 掌握度评分两维与档位。
> 本技能会**写**:新增一份 review-session 记录，并更新 progress.json 与 knowledge-tree.md。

---

## 核心原则

1. **掌握度由系统独立评估，绝不让用户自评。** 用一个没参与对话的子智能体打分，避开「考官心软」偏差。
2. **覆盖度 + 深度两维并重。** 会背（覆盖广但浅）和钻牛角尖（深但窄）都不算掌握，见 mastery-rubric。
3. **进度只增不乱。** 严格按 data-contract 更新 progress，艾宾浩斯算 nextReview，状态机别跳。
4. **评分要有理由。** review-session 必须写清「覆盖了什么、深度如何、缺什么」。

## 借口对照表(出现这些念头=正在偷懒,照右栏纠正)

| 借口 | 现实 |
|---|---|
| 「用户自评 8 分,就记 8 分」 | 用户自评天然偏高;只作事实输入,分数不采信。掌握度由子智能体按 rubric 客观给。 |
| 「会话很短,我自己打分就行,不必派子智能体」 | 越短越主观。无论长短,阶段 2 必须走 Agent 工具评估。 |
| 「Agent 调用失败,那我自己打吧」 | 禁止回退自评。失败则暂停归档、提示用户重试 `/swcc-stop`。 |
| 「这概念讲解时用户反应不错,算掌握」 | 只有考核阶段算数;只讲没考的概念不评分、不记薄弱。 |
| 「答错但意思差不多,算对」 | 按 rubric 硬标准,不给人情分。 |
| 「薄弱点抄上次的,不重新评」 | 每次都重评并更新 weakPoints 的 mastery/consecutivePass/nextReview。 |

## Red Flags(自查,出现即停下)

- 正准备采信用户报的分数 → 必须派子智能体。
- 正准备把讲解阶段表现计入评分/薄弱点 → 只看考核阶段。
- 正因「会话短」或「Agent 失败」想自己打分 → 派子智能体;失败则暂停归档。
- 给某维度 ≤3 却让 mastery >5 → 违反 rubric 约束。
- 改了 progress.json 却没同步 knowledge-tree 的勾选/🔴 → 必须一致。

## 常见错误

1. **直接采信 go 收尾小结里的分数**:小结是 go 自写、带主观。→ 子智能体要基于原始问答重评。
2. **两个计数混用**:节点 `reviewCount`(复习次数,用于艾宾浩斯)≠ 薄弱点 `consecutivePass`(连续达标,用于移除)。
3. **nextReview 基准算错**:节点和薄弱点要各用**自己的** reviewCount 取间隔。
4. **忘记同步 tree**:progress 改了,knowledge-tree 的 `[x]`/🔴 没跟上。

---

## 执行流程

| 阶段 | 名称 | 目的 |
|---|---|---|
| 1 | 提取本次复习 | 从对话（含 go 收尾小结）整理事实 |
| 2 | **独立子智能体评分** | 客观给 coverage/depth/mastery |
| 3 | 写复习记录 | review-sessions/<时间戳>.md |
| 4 | 更新 progress.json | status/mastery/nextReview/weakPoints/stats |
| 5 | 更新 knowledge-tree.md | 勾选已掌握、标薄弱点 |
| 6 | 输出摘要 | 掌握度 + 理由 + 下次复习 |

---

### 阶段 1：提取本次复习内容

从**当前对话历史**回顾本次（通常由 `/swcc-go` 发起的）复习，整理出事实清单:
- 复习了哪个专题、哪个节点路径、什么 mode;
- 该节点的核心概念清单（从 knowledge-system.md 取）;
- 问了哪些题、用户怎么答、哪里答得好 / 哪里卡壳或答错;
- 若 go 输出过「收尾小结」，把它一并纳入。

> **关键:区分「讲解阶段」与「考核阶段」。** go 默认先讲后测——**评分与薄弱点只看考核阶段**的表现。光在讲解阶段过了一遍、还没考的概念**不算薄弱点**(用户只是刚复习,不是不会);只有「讲过/复习过、考核时仍卡壳或答浅」的才记薄弱点。整理事实时把这两类分清,交给阶段 2 的评估子智能体。
>
> **怎么区分两阶段(按可靠性从高到低)**:① **找 go 输出的硬分界标记 `--- 考核开始 ---`**——它**之后**算考核、之前算讲解,这是**首选且最可靠**的依据;② 有 go 收尾小结的「考核结果」表 → 直接以该表为考核事实来源(每行的「表现」=扎实/答浅/卡壳);③ 都没有(go 没走完或漏标)才回退启发式:`test` 取向全程算考核,`study` 取向把「明确提问→等答→点评对错」的往返算考核。**注意**:讲解阶段 check-in 的「这块清楚吗」式确认/互动**不是考核**,绝不据此记薄弱点。

若对话里找不到本次复习内容（比如没先 `/swcc-go`），告诉用户「本轮对话没有可归档的复习」，停止。

### 阶段 2：派独立子智能体评估掌握度（不让用户自评）

**这是客观性的关键，必做。** 用 **Agent 工具派一个评估子智能体**，把阶段 1 整理的事实清单 + `mastery-rubric.md` 的评分规则交给它，要求它**只评估、不写文件**，按 rubric 第五节返回 JSON:

```json
{ "mastery": 7, "coverage": 6, "depth": 8, "reasons": "…", "weakPoints": [ { "node": "<节点路径>", "concept": "…", "mastery": 4 } ] }
```

> 评分以评估子智能体返回的结果为准，不要用主对话的主观印象覆盖它。若返回结果与对话中的事实明显矛盾，可要求其复核一次；否则采信。

- **用户在对话里自评分数**(如「我觉得有 8 分」「按 8 分记」):把它当"用户自我感知"记入事实,但**子智能体不采信该分数**,仍按 rubric 客观评。
- **Agent 工具调用失败**:**绝不回退到主对话自己打分**。暂停归档,告诉用户「评估暂不可用,本次未写入,请稍后重试 `/swcc-stop`」,然后停止。

### 阶段 3：写复习记录

在 `$HOME/.study-with-cc/topics/<slug>/review-sessions/` 下新建 `<时间戳>.md`（时间戳用 `date +%Y-%m-%d-%H-%M-%S`）:

```markdown
# 复习记录：<YYYY-MM-DD HH:mm>

## 元信息
- 专题：<topic>
- 节点路径：<节点路径>
- 模式：<mode>
- 掌握度（系统评估）：<mastery>/10（覆盖度 <coverage> / 深度 <depth>）

## 评分理由
<子智能体的 reasons：覆盖了什么、深度如何、缺什么>

## 复习内容
- 问题 1：…（回答要点 / 点评）
- 问题 2：…（卡壳）

## 薄弱点
- <具体薄弱点，若有>

## 下次复习建议
- <YYYY-MM-DD> 复习：<重点>
```

### 阶段 4：更新 progress.json

- 目标节点:`status`（mastery ≥ 8 → `mastered`，否则 `in_progress`）、`mastery`、`lastReviewed`=今天（`date +%F`）、`reviewCount += 1`、按艾宾浩斯（data-contract 第六节）用**该节点的 reviewCount** 算 `nextReview`。
- **薄弱点（只来自考核阶段）**:把子智能体返回的 `weakPoints` + 考核中卡壳/答错/掌握度低（<6）的点写入/更新 `weakPoints`。子智能体只给出 `node/concept/mastery`(候选),**其余字段由 stop 主体补齐**:
  - **新薄弱点**(weakPoints 里原本没有):`createdAt`=今天、`reviewCount`=0、`consecutivePass`=0、`nextReview` 按 `reviewCount`(本次 +1→1)算。
  - **已存在的薄弱点**:**继承其旧的 `reviewCount`/`consecutivePass`**,绝不重置为 0;再按下面规则更新——本次考核 `mastery ≥ 8` → `consecutivePass += 1`、否则 `consecutivePass = 0`;`reviewCount += 1`;`nextReview` 用**该薄弱点自身**(更新后的) `reviewCount` 算。**`consecutivePass ≥ 2` 时移除该薄弱点。**
- `stats`:`totalReviewCount += 1`;`totalReviewTime` 加本次估算时长（分钟;**粗估 = 本轮你在讲解+考核里的回复条数 × 2**,有真实时间信息时以实际为准;别留空、别乱填）;**全局 `streak`**（`lastReviewDate` 昨天→+1、今天→不变、否则→重置 1）;`lastReviewDate`=今天。
- `currentNode` 推进:本次 mastery ≥ 8 → 推进到下一个 `not_started` 叶子（先同子主题、再下个子主题）;mastery < 8 → 保持该节点;用户明确要求换下一个 → 推进但该节点保持 `in_progress`。

### 阶段 5：更新 knowledge-tree.md

- 目标节点已 `mastered` → 对应行 `- [ ]` 改 `- [x]`。
- 新产生薄弱点 → 对应行尾加 `🔴`（已有则保持）;若某行薄弱点已清除则去掉 `🔴`。

> 全部落盘成功后,**删除 cwd 的 `./.swcc-checkin.md`**(本轮闭环结束,go 的深挖草稿作废;见 data-contract 第十五节)。文件不存在则忽略,不报错。

### 阶段 6：输出摘要

```
✅ 复习已归档
📊 掌握度（系统评估）：<mastery>/10（覆盖 <coverage> / 深度 <depth>）—— <一句话理由>
🔴 新增薄弱点：<列表，或「无」>
🗑️ 已移除薄弱点（连续两次达标）：<列表，或「无」>
📅 下次复习：<YYYY-MM-DD>
🔥 连续复习：<streak> 天
📁 记录：~/.study-with-cc/topics/<slug>/review-sessions/<时间戳>.md
```

---

## 质量基准（达到才算归档完成）

- 掌握度来自**独立子智能体**，不是主对话自评;review-session 里有可追溯的评分理由。
- progress.json 的 mastery/status/nextReview/weakPoints/stats 全部按 rubric 与 data-contract 更新一致。
- knowledge-tree.md 的勾选与 🔴 标记和 progress 对得上。
