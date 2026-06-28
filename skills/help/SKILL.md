---
name: swcc-help
description: Use when the user asks how to use this plugin, what the commands do, where to start, which command fits a goal, or wants an overview/cheatsheet of study-with-claude-code. 触发:「swcc 怎么用」「help」「这插件能干啥」「我该用哪个命令」「从哪开始」「命令列表」「使用说明」。
argument-hint: "[可选：你想做的事 / 某个命令名]"
user-invocable: true
---

# swcc · help — 使用说明

用户想了解怎么用这套复习插件。**把下面的指南呈现给用户**;若 `$ARGUMENTS` 指向某个具体问题或命令(如"怎么开始"、"go 和 stop 区别"、"mock"),就**聚焦回答那一点**,不必全量铺开。

---

## 一句话

把"看完就忘"的学习,变成一个会复利的闭环:**建知识树 → 每天看该复习啥 → 先讲后测地复习 → 系统客观评分 → 复盘让系统越来越懂你**。任何专题都能用,数据存在 `~/.study-with-cc/`。

## 核心心智模型

```
/swcc-plan   建/更新某专题的知识树（一次）
   ↓
/swcc-daily  每天看今天该复习什么（跨专题）
   ↓
/swcc-go → 答题 → /swcc-stop     一次复习闭环（先讲后测 → 归档评分）
   ↓
/swcc-compound   阶段复盘，提炼"你怎么学"，让以后讲解更贴你
```
⚠️ **`go`/`weak` 必须和 `stop` 在同一轮对话里**——stop 要读本次对话来评分归档。

## 「我想…」→ 用哪个

| 我想… | 用 |
|---|---|
| 开始准备一个新专题 | `/swcc-plan <主题> [p6/p7/p8/p9]` |
| 给已有专题补充/加深 | `/swcc-plan <主题> --update` |
| 知道今天该复习什么 | `/swcc-daily` |
| 系统地学+被考一个专题 | `/swcc-go` → 答题 → `/swcc-stop` |
| 只攻我的薄弱点 | `/swcc-weak` → 答题 → `/swcc-stop` |
| 直接被考、不用先讲 | `/swcc-go <主题> test` |
| 模拟一场面试 | `/swcc-mock` |
| 看进度/掌握比例/薄弱点 | `/swcc-stats` |
| 在多个专题间切换 / 看总览 | `/swcc-switch` |
| 把某概念横向深挖 | `/swcc-deep <主题> <概念>` |
| 复盘、让系统更懂我的学法 | `/swcc-compound` |
| 把今天学的写成 blog | `/swcc-blog` |

## 全部命令

**核心闭环**:`plan`(建树) · `go`(先讲后测复习) · `stop`(归档评分) · `stats`(进度快照)
**日常 & 多专题**:`daily`(今天复习啥) · `switch`(切换/总览) · `weak`(薄弱点专项)
**进阶 & 沉淀**:`mock`(模拟面试) · `compound`(复盘+让系统懂你) · `deep`(概念深挖) · `blog`(写成文章)

> 任何命令不带参数都能用:topic 缺省用当前专题,其它走默认。

## 几个容易卡的点

- **先 `plan` 再用别的**:没有知识树,go/stats 等无从下手。
- **`go`/`weak` 配 `stop`、且同轮对话**:否则这次复习不会被评分归档。
- **掌握度不是自评**:由独立子智能体按"覆盖度+深度"客观打;薄弱点只从**考核阶段**记(讲解阶段不算)。
- **`compound` 不是统计**(统计看 `stats`):它和你聊学习感受、提炼"怎么教你",沉淀进 `learner-profile.md`,让后续讲解更贴你。
- **`blog` 写到当前目录 `./blogs/`**(方便发布);其它数据都在 `~/.study-with-cc/`,更新插件不丢。

## 典型节奏

```
第一次：   /swcc-plan <主题> p7
每天：     /swcc-daily → /swcc-go → 答题 → /swcc-stop
攻弱项：   /swcc-weak → 答题 → /swcc-stop
阶段复盘： /swcc-compound      （隔段时间一次）
面试前：   /swcc-mock          沉淀：/swcc-blog
```

> 升级到最新版:`claude plugin update study-with-claude-code`。
