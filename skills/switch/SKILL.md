---
name: swcc-switch
description: Use when the user wants to switch the active topic, list their topics, or see a cross-topic overview. 触发:「切换到 X」「切到 X」「列出专题」「我有哪些专题」「switch」「换个专题」「当前在学啥」。
argument-hint: "[topic]"
user-invocable: true
---

# swcc · switch — 多专题切换 + 总览

切换当前默认专题,或在无参数时列出所有专题的总览供选择。

> 开始前先读数据契约:`${CLAUDE_PLUGIN_ROOT}/skills/_shared/data-contract.md`(第九节 config、第十三节到期判定)。

参数:`$ARGUMENTS` —— 可选 topic(目标专题的名称或 slug)。

---

## 核心原则

1. **唯一写入 = `config.json` 的 `activeTopic`。** 不碰任何专题的 tree / progress / system——总览部分全是只读。
2. **不存在就别乱写。** 目标专题不存在时,列出可选项让用户重选,绝不新建或瞎指。
3. **总览要给决策信息。** 列专题时带掌握比例 + 今日到期数,让用户一眼看出该去哪。

---

## 流程

### 1. 读取专题表
- 读 `config.json` 的 `topics[]` 与当前 `activeTopic`。config 不存在/缺失 → 按数据契约第九节兜底扫 `topics/`;一个都没有 → 提示先 `/swcc-plan <topic>`,停止。

### 2. 分支:带参 or 无参

| 情况 | 行为 |
|---|---|
| **带 topic 参数** | 把参数规整为 slug,在 `topics[]` + 磁盘 `topics/<slug>/` 里匹配。匹配到 → 设 `activeTopic` = 该 slug,写回 `config.json`,确认。匹配不到 → 列出现有专题让用户重选,**不写**。 |
| **无参数** | 渲染总览(下方),让用户选一个 → 设 `activeTopic` 写回。 |

### 3. 总览渲染（只读）
对每个专题,读其 `progress.json` 算:已掌握/总数比例、今日到期数(数据契约第十三节)、上次复习日(`stats.lastReviewDate`)。
```
📚 我的复习专题
━━━━━━━━━━━━━━━━━━━━
▶ <topic-A>   🟢 <a>/<total> 掌握   🔔 今日到期 <n>   上次 <date>   ← 当前
  <topic-B>   🟢 <a>/<total> 掌握   🔔 今日到期 <n>   上次 <date>
切换:/swcc-switch <topic>
```

### 4. 切换后确认
```
✅ 已切换到 <topic>（activeTopic）
下一步:/swcc-go 开始复习，或 /swcc-daily 看今天到期。
```

---

## 质量基准
- 只改了 `config.json` 的 `activeTopic` 一个字段,其余文件未动。
- 目标不存在时未误写、给了可选项。
- 总览的比例/到期数与各 progress.json 一致。
