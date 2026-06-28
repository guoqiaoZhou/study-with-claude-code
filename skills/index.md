---
name: swcc
description: 展示 study-with-claude-code 插件的全部技能清单与典型用法。Use this skill when the user asks what this plugin can do, how to use swcc, or wants the command list.
user-invocable: true
---

# study-with-claude-code · 技能索引

一款 AI 驱动的通用复习系统：建知识树 → 闭环复习 → 自动评估掌握度 → 按艾宾浩斯曲线安排复习。数据存在 `~/.study-with-cc/`。

## 核心闭环

| 技能 | 调用 | 作用 |
|---|---|---|
| plan | `/swcc-plan [topic] [level] [--update]` | 生成 roadmap 级知识树 + 知识体系（可挂 PDF/文档），生成后用评审子智能体查漏补缺;对已有专题可增量更新、保留进度 |
| go | `/swcc-go [topic] [study\|test] [concept\|scenario\|mixed]` | 开始复习：先带你过一遍知识点，再苏格拉底式考核（先讲后测；优先到期薄弱点） |
| stop | `/swcc-stop` | 结束并归档：独立子智能体客观评估掌握度、记薄弱点、更新进度 |
| stats | `/swcc-stats [topic]` | 看进度：掌握比例、薄弱点、连续天数、统计 |

## 日常 & 多专题

| 技能 | 调用 | 作用 |
|---|---|---|
| daily | `/swcc-daily [topic]` | 今天该复习什么：跨专题汇总到期项 + 建议（只读，适合定时提醒） |
| switch | `/swcc-switch [topic]` | 多专题切换 + 总览（掌握比例 + 今日到期） |
| weak | `/swcc-weak [topic]` | 薄弱点专项强化（到期优先；配 `/swcc-stop` 归档） |

> `go` / `weak` 与 `stop` 必须在**同一轮对话**里使用——stop 要读本次对话来归档。

## 进阶

| 技能 | 调用 | 作用 |
|---|---|---|
| mock | `/swcc-mock [topic] [level]` | 模拟面试：跨节点连续出题 + 独立子智能体评分报告（偏向薄弱点） |
| compound | `/swcc-compound [topic]` | 阶段沉淀报告：聚合历次记录看趋势，写入 reports/ |
| deep | `/swcc-deep [topic] [concept]` | 概念深挖：跨领域连接/类比/对比/延伸（可选联网），沉淀 deep-notes/ |
| blog | `/swcc-blog [拆分提示] [out:目录]` | 把本轮学习/讨论沉淀成可读 blog 文章（默认写到 ./blogs/，可发布） |

## 典型用法

```
首次建专题：  /swcc-plan <主题> p7      （可挂载相关 PDF/资料）
每天开始：    /swcc-daily              （看今天到期什么）
日常复习：    /swcc-go  →  答题  →  /swcc-stop
攻克弱项：    /swcc-weak  →  答题  →  /swcc-stop
查看进度：    /swcc-stats　切换专题：/swcc-switch
模拟面试：    /swcc-mock　阶段报告：/swcc-compound　深挖概念：/swcc-deep <concept>
沉淀成文：    /swcc-blog               （把本轮学习/讨论写成 blog，存 ./blogs/）
```

## 数据位置

所有复习数据在 `~/.study-with-cc/`，独立于插件目录，插件更新不会丢。参考资料只记路径不复制（见各技能共享的 `_shared/data-contract.md`）。
