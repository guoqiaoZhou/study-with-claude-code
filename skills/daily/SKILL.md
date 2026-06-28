---
name: swcc-daily
description: Use when the user wants to know what to review today, or wants a daily review reminder/digest across topics. 触发:「今天复习什么」「该复习啥了」「daily」「复习提醒」「今天有什么要复习」「到期了哪些」;也用于定时(cron/loop)每日唤起。
argument-hint: "[topic]"
user-invocable: true
---

# swcc · daily — 今天该复习什么

只读地汇总**今天到期该复习**的内容,给出建议,指向 go / weak 去执行。**零写入**——正因如此适合定时无人值守地触发。

> 开始前先读数据契约:`${CLAUDE_PLUGIN_ROOT}/skills/_shared/data-contract.md`(第十三节 `due(today)` 与 dailyGoal)。本技能**只读不写**。

参数:`$ARGUMENTS` —— 可选 topic;**缺省时跨所有专题汇总**(每日提醒最有用)。

---

## 核心原则

1. **只读,零写入。** 不改任何文件——这样定时触发也绝不会误伤数据。
2. **不编数字。** 没有到期项就如实说"今天没有到期";数据不足的不臆造。
3. **最该复习的排最前。** 一律按 `nextReview` 升序(逾期最久在前)。
4. **给可执行的下一步。** 每项指向 `/swcc-weak` 或 `/swcc-go <topic>`,让用户一键接着做。

---

## 流程

### 1. 收集到期项
- 带 topic → 只看该专题;缺省 → 读 `config.json` 的 `topics[]`,逐个读其 `progress.json`。
- 按数据契约第十三节算 `due(today)` = `nextReview ≤ 今天(date +%F)` 的**节点 + 薄弱点**(`nextReview` 为 null 的不算)。
- config 不存在/缺失 → 按数据契约第九节兜底扫 `topics/`;一个专题都没有 → 提示先 `/swcc-plan <topic>`,停止。

### 2. 排序与分组
- 每个专题内:到期项按 `nextReview` 升序;薄弱点和节点都计入,标注逾期天数。
- 跨专题:逐专题列出其到期数。

### 3. 算 dailyGoal 进度
- 从今天的 `review-sessions/` 估"已复习时长",对照 `config.reviewSettings.dailyGoal`(分钟)。数据不足只显示目标。

### 4. 渲染
```
🔔 今天该复习（<date +%F>）
━━━━━━━━━━━━━━━━━━━━
<topic-A>：到期 <n>（薄弱点 <w>）
  · <概念/节点>        逾期 <d> 天   → /swcc-weak <topic-A>
  · <概念/节点>        今天到期       → /swcc-go <topic-A>
<topic-B>：到期 <n>
  · …

今日目标 <goal>min ｜ 已复习 <done>min
建议:<一句话——先清逾期最久的,再过到期薄弱点>
```
- 全部专题都没有到期项 → 输出「今天没有到期要复习的,可以 `/swcc-go <topic>` 学新内容,或 `/swcc-stats` 看进度」。

---

## 质量基准
- 到期判定严格按 `nextReview ≤ 今天`,`null` 不计;排序正确(逾期久的在前)。
- 全程只读,未写任何文件。
- 每条到期项都给了可点的下一步命令。
