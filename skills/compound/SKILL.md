---
name: swcc-compound
description: Use when the user wants a periodic consolidation report showing review trends over time, not the current snapshot. 触发:「复习报告」「阶段总结」「沉淀」「compound」「这段时间复习得怎么样」「生成报告」「学习报告」。
argument-hint: "[topic]"
user-invocable: true
---

# swcc · compound — 阶段沉淀报告

把一段时间的复习记录聚合成**趋势报告**并落盘。与 `stats`(当前快照)不同,compound 看的是**随时间的变化**,并给下阶段建议。

> 开始前先读数据契约:`${CLAUDE_PLUGIN_ROOT}/skills/_shared/data-contract.md`(尤其第十二节大文件分块写)。本技能**只读聚合**复习记录,只**写**一份 `reports/` 报告,不改 progress。

参数:`$ARGUMENTS` —— 可选 topic(默认 activeTopic)。

---

## 核心原则

1. **趋势,不是快照。** 必须体现"随时间变化"(掌握度曲线、薄弱点增减),否则就是 stats。
2. **只读聚合 + 只写报告。** 读 `review-sessions/` + `mock-sessions/` + `progress.json`,只写 `reports/<date>.md`;**不改 progress/tree**。
3. **不编数字。** 记录不足就如实说明、少画趋势,绝不臆造曲线。
4. **报告分块写。** 按数据契约第十二节先骨架后分段,避免超时。

---

## 流程

### 1. 收集
- topic 缺省 → activeTopic(缺失按数据契约第九节兜底)。
- 读该专题 `review-sessions/` + `mock-sessions/` 全部记录(每份含日期、mastery/coverage/depth、薄弱点)、当前 `progress.json`。
- 记录少于 2 份 → 告诉用户"复习记录太少,趋势意义不大,可先多复习几次或用 `/swcc-stats` 看当前进度",可停止或只给简版。

### 2. 聚合
- **掌握度趋势**:按日期把各次 mastery 排序,形成时间序列。
- **薄弱点趋势**:各时间点的 weakPoints 数量(新增/消除)。
- **节点分布**:当前 mastered/in_progress/not_started 比例(取自 progress)。
- **总量**:总复习次数、总时长、最长 streak(取自 stats + 记录)。

### 3. 写报告（分块,reports/<date +%F>.md）
先骨架,再逐段 Edit 追加:
```markdown
# <topic> 复习报告（<起始日> ~ <today>）
## 概览
- 总复习次数：<n>　总时长：<t> 分钟　最长 streak：<d> 天
## 掌握度趋势
<date>: <ASCII 条 或 百分比>
...
## 知识图谱
🟢已掌握 <a> / 🟡学习中 <b> / 🔴未开始 <c>
## 薄弱点趋势
新增：… ｜ 消除：…
## 下阶段建议
1. <进入哪个新主题 / 该加强什么 / 模拟面试频率>
```

### 4. 输出摘要
```
📈 <topic> 阶段报告已生成
　总复习 <n> 次 ｜ 掌握度 <最早>→<最新> ｜ 薄弱点 <最早数>→<最新数>
💡 下阶段：<一句话建议>
📁 报告：~/.study-with-cc/topics/<slug>/reports/<date>.md
```

---

## 质量基准
- 报告体现了时间维度的变化(不是 stats 式快照);数据不足处如实留白。
- 只写了 reports/ 一份文件,progress/tree 未动。
- 长报告分块写、未超时。
