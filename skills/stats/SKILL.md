---
name: swcc-stats
description: 展示一个专题的复习进度——掌握/学习中/未开始比例、薄弱点清单、连续复习天数和总量统计。Use this skill when the user wants to see review progress/stats/dashboard, or says "看进度"、"stats"、"复习到哪了"、"进度如何"。
argument-hint: "[topic]"
user-invocable: true
---

# swcc · stats — 复习进度展示

只读地展示一个专题的复习进度。

> 开始前先读数据契约：`${CLAUDE_PLUGIN_ROOT}/skills/_shared/data-contract.md`。本技能**只读不写**。

参数：`$ARGUMENTS` —— 可选 topic（默认用 config 的 `activeTopic`）。

## 流程

### 1. 加载数据
- topic 缺省 → 读 `$HOME/.study-with-cc/config.json` 的 `activeTopic`（config 不存在或缺 activeTopic 时，按数据契约第九节「兜底」扫 topics/ 自愈，不要直接报错）。
- 该专题不存在 → 提示先 `/swcc-plan <topic>`，停止。
- 读 `progress.json`（必要时读 `review-sessions/` 做趋势聚合）。

### 2. 计算
- 按 `nodes` 的 `status` 统计：mastered / in_progress / not_started 各占多少、百分比。
- `weakPoints` 按 `nextReview` 升序排列（最该复习的在前）。
- 从 `stats` 取 streak、totalReviewCount、totalReviewTime。

### 3. 渲染
```
📊 <topic> 复习进度
━━━━━━━━━━━━━━━━━━━━
🟢 已掌握：<a> / <total> (<x%>)
🟡 学习中：<b> / <total> (<y%>)
🔴 未开始：<c> / <total> (<z%>)

🔴 薄弱点（<n>）
1. <concept>（节点：<node>　上次：<lastReviewed>　下次：<nextReview>）
2. …

🔥 连续复习：<streak> 天
🔢 总复习次数：<totalReviewCount>
⏱️ 总复习时长：<totalReviewTime> 分钟
```

掌握度变化趋势（可选）：若 `review-sessions/` 有多份记录，可按日期聚合各节点掌握度，画一个简单的 ASCII 趋势条；数据不足时省略此段，不要编造数字。
