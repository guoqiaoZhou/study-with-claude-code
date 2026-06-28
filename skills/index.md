---
name: index
description: 展示 study-with-claude-code 插件的全部技能清单与典型用法。Use this skill when the user asks what this plugin can do, how to use in-prep, or wants the command list.
user-invocable: true
---

# study-with-claude-code · 技能索引

一款 AI 驱动的通用复习系统：建知识树 → 闭环复习 → 自动评估掌握度 → 按艾宾浩斯曲线安排复习。数据存在 `~/.in-prep/`。

## 一期技能（闭环四件套）

| 技能 | 调用 | 作用 |
|---|---|---|
| plan | `/study-with-claude-code:plan [topic] [level]` | 为专题生成知识树 + 知识体系，可挂载 PDF/文档作依据 |
| go | `/study-with-claude-code:go [topic] [mode]` | 开始复习：自动定位知识点，逐题考核（优先到期薄弱点） |
| stop | `/study-with-claude-code:stop` | 结束并归档：系统评估掌握度、记薄弱点、更新进度 |
| stats | `/study-with-claude-code:stats [topic]` | 看进度：掌握比例、薄弱点、连续天数、统计 |

> `go` 与 `stop` 必须在**同一轮对话**里使用——stop 要读本次对话来归档。

## 典型用法

```
首次建专题：  /plan JVM p7        （可挂载《深入理解JVM》PDF）
日常复习：    /go  →  答题  →  /stop
查看进度：    /stats
```

## 规划中（后续阶段）

`weak`（薄弱点强化）、`mock`（模拟面试）、`deep`（跨领域深挖）、`compound`（阶段沉淀报告）、`switch`（多专题切换）。

## 数据位置

所有复习数据在 `~/.in-prep/`，独立于插件目录，插件更新不会丢。参考资料只记路径不复制（见各技能共享的 `_shared/data-contract.md`）。
