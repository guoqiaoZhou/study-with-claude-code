---
name: swcc-stop
description: 结束本次复习并归档。从当前对话提取复习内容，由系统评估掌握度（不让用户自评），记录薄弱点，更新进度并按艾宾浩斯曲线安排下次复习。Use this skill when the user wants to end/finish/archive a review session, or says "结束复习"、"stop"、"归档"、"复习完了"。
user-invocable: true
---

# swcc · stop — 结束复习并归档

归档**当前这轮对话里**刚发生的复习：评分、记薄弱点、更新进度。

> 开始前先读数据契约：`${CLAUDE_PLUGIN_ROOT}/skills/_shared/data-contract.md`。本技能会**写**：新增一份 review-session 记录，并更新 progress.json 与 knowledge-tree.md。

## 流程

### 1. 提取本次复习内容
从**当前对话历史**回顾本次（通常由 `/swcc-go` 发起的）复习：
- 复习了哪个专题、哪个节点路径；
- 问了哪些题、用户怎么答的、哪里答得好 / 哪里卡壳或答错。

若无法从对话中找到本次复习内容（比如没先 `/swcc-go`），告诉用户「本轮对话没有可归档的复习」，停止。

### 2. 系统评估掌握度（不让用户自评）
**掌握度（1–10）完全由你根据用户在本次对话中的表现评估**，主要参考两个维度：
1. **知识面覆盖度** —— 本次实际触及该节点多少关键概念（覆盖全 vs 只碰到一角）。
2. **回答深度** —— 用户是停留在背诵/罗列，还是讲清了原理、设计取舍、边界条件、能举例。

给出分数的同时，**写明评分理由**（覆盖了什么、深度如何、缺什么）。不要询问用户自评。

### 3. 写复习记录
在 `$HOME/.study-with-cc/topics/<slug>/review-sessions/` 下新建 `<时间戳>.md`（时间戳用 `date +%Y-%m-%d-%H-%M-%S`）：

```markdown
# 复习记录：<YYYY-MM-DD HH:mm>

## 元信息
- 专题：<topic>
- 节点路径：<节点路径>
- 模式：<mode>
- 掌握度（系统评估）：<n>/10

## 评分理由
<覆盖度 + 深度的具体说明>

## 复习内容
- 问题 1：…（回答要点 / 点评）
- 问题 2：…（卡壳）

## 薄弱点
- <具体薄弱点，若有>

## 下次复习建议
- <YYYY-MM-DD> 复习：<重点>
```

### 4. 更新 progress.json
- 目标节点：更新 `status`（mastery ≥ 8 → `mastered`，否则 `in_progress`）、`mastery`、`lastReviewed`（今天，`date +%F`）、`reviewCount += 1`、按艾宾浩斯（数据契约第六节）算 `nextReview`。
- **薄弱点**：本次卡壳/答错/掌握度低（< 6）的具体点，写入 `weakPoints`（含 node/concept/mastery/createdAt/nextReview/reviewCount）。若某薄弱点本次掌握度 ≥ 8 且此前已达标一次，则移除它。
- `stats`：`totalReviewCount += 1`；`totalReviewTime` 加上本次估算时长（分钟，可据对话规模估）；更新 `streak`（若 `lastReviewDate` 是昨天则 +1，是今天则不变，否则重置为 1）；`lastReviewDate` = 今天。
- 若需要把 `currentNode` 推进到下一个 `not_started` 节点，则更新它。

### 5. 更新 knowledge-tree.md
- 若目标节点已 `mastered` → 把对应行的 `- [ ]` 改成 `- [x]`。
- 若新产生薄弱点 → 对应行尾加 `🔴`（已是薄弱点则保持）。

### 6. 输出摘要
```
✅ 复习已归档
📊 掌握度（系统评估）：<n>/10 —— <一句话理由>
🔴 新增薄弱点：<列表，或「无」>
📅 下次复习：<YYYY-MM-DD>
🔥 连续复习：<streak> 天
📁 记录：~/.study-with-cc/topics/<slug>/review-sessions/<时间戳>.md
```
