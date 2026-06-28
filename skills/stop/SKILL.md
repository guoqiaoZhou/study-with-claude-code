---
name: swcc-stop
description: Use when the user wants to end and archive the current review session — must be the same conversation as the go/weak session it closes. 触发:「结束复习」「stop」「归档」「复习完了」「今天就到这」。
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

若对话里找不到本次复习内容（比如没先 `/swcc-go`），告诉用户「本轮对话没有可归档的复习」，停止。

### 阶段 2：派独立子智能体评估掌握度（不让用户自评）

**这是客观性的关键，必做。** 用 **Agent 工具派一个评估子智能体**，把阶段 1 整理的事实清单 + `mastery-rubric.md` 的评分规则交给它，要求它**只评估、不写文件**，按 rubric 第五节返回 JSON:

```json
{ "mastery": 7, "coverage": 6, "depth": 8, "reasons": "…", "weakPoints": [ { "concept": "…", "mastery": 4 } ] }
```

> 评分以评估子智能体返回的结果为准，不要用主对话的主观印象覆盖它。若返回结果与对话中的事实明显矛盾，可要求其复核一次；否则采信。

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

- 目标节点:`status`（mastery ≥ 8 → `mastered`，否则 `in_progress`）、`mastery`、`lastReviewed`=今天（`date +%F`）、`reviewCount += 1`、按艾宾浩斯（data-contract 第六节）算 `nextReview`。
- **薄弱点**:子智能体返回的 `weakPoints` + 本次卡壳/答错/掌握度低（< 6）的具体点，写入 `weakPoints`（含 node/concept/mastery/createdAt/nextReview/reviewCount）。若某薄弱点本次 mastery ≥ 8 且此前已达标一次，则移除它（连续两次达标才移除）。
- `stats`:`totalReviewCount += 1`;`totalReviewTime` 加本次估算时长（分钟，据对话规模估）;`streak`（`lastReviewDate` 是昨天则 +1，今天则不变，否则重置 1）;`lastReviewDate`=今天。
- 视情况把 `currentNode` 推进到下一个 `not_started` 节点。

### 阶段 5：更新 knowledge-tree.md

- 目标节点已 `mastered` → 对应行 `- [ ]` 改 `- [x]`。
- 新产生薄弱点 → 对应行尾加 `🔴`（已有则保持）;若某行薄弱点已清除则去掉 `🔴`。

### 阶段 6：输出摘要

```
✅ 复习已归档
📊 掌握度（系统评估）：<mastery>/10（覆盖 <coverage> / 深度 <depth>）—— <一句话理由>
🔴 新增薄弱点：<列表，或「无」>
📅 下次复习：<YYYY-MM-DD>
🔥 连续复习：<streak> 天
📁 记录：~/.study-with-cc/topics/<slug>/review-sessions/<时间戳>.md
```

---

## 质量基准（达到才算归档完成）

- 掌握度来自**独立子智能体**，不是主对话自评;review-session 里有可追溯的评分理由。
- progress.json 的 mastery/status/nextReview/weakPoints/stats 全部按 rubric 与 data-contract 更新一致。
- knowledge-tree.md 的勾选与 🔴 标记和 progress 对得上。
