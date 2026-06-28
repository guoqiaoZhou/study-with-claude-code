---
name: go
description: 开始一次专题复习。自动定位当前该复习的知识点（优先到期薄弱点），结合知识体系与挂载的参考资料逐题考核。Use this skill when the user wants to start reviewing, studying, or practicing a topic, or says "开始复习"、"go"、"复习一下 JVM"、"考考我"。
argument-hint: "[topic] [mode]"
user-invocable: true
---

# in-prep · go — 开始复习

发起一次复习会话。**这是一次对话内的复习**：本技能负责加载上下文并逐题考核，复习结束后用 `/stop` 归档。

> 开始前先读数据契约：`${CLAUDE_PLUGIN_ROOT}/skills/_shared/data-contract.md`。本技能**只读不写**——所有进度/掌握度的落盘由 `/stop` 完成。

参数：`$ARGUMENTS` —— 可选 topic（默认用 config 的 activeTopic）、可选 mode（`concept`/`scenario`/`mixed`，默认 `mixed`）。

## 流程

### 1. 解析参数 & 加载专题
- topic 缺省 → 读 `$HOME/.in-prep/config.json` 的 `activeTopic`。
- 若该专题不存在 → 提示用户先用 `/plan <topic>` 建知识树，停止。
- 读该专题的 `progress.json`、`knowledge-tree.md`、`knowledge-system.md`、`references.json`。

### 2. 定位当前复习节点
按优先级：
1. `weakPoints` 中 `nextReview` ≤ 今天（`date +%F`）的薄弱点 —— 优先复习到期薄弱点；
2. 否则 `currentNode` 指向的节点（若为 `in_progress`）；
3. 否则下一个 `not_started` 的叶子节点。

### 3. 取出该节点内容
- 从 `knowledge-system.md` 取该节点的核心概念与面试考察点。
- **若 `references.json` 有资料**：按数据契约第八节，**只读与当前节点相关的章节/页**（PDF 分页读），作为讲解与出题依据。不要全文导入。

### 4. 宣布并开始考核
先打印一行定位信息：
```
🎯 当前复习：<topic> / <节点路径>
📅 上次复习：<lastReviewed 或「首次」>
🔄 模式：<mode>
```
然后**一次只问一道题**：
- 提问 → 等用户回答 → 针对回答**点评**（指出对/错/缺漏，必要时补讲原理与取舍）→ 再问下一题。
- 题目结合 mode：`concept` 偏原理、`scenario` 给实际场景、`mixed` 交替。
- 围绕当前节点充分考核（覆盖其关键概念），不要一次跳太多节点。

### 5. 收尾提示
当用户表示告一段落（或你判断本节点已考核充分）时，提醒：
```
复习完用 /stop 归档本次记录（掌握度由系统评估，会自动记录薄弱点并安排下次复习）。
⚠️ /go 和 /stop 需在同一轮对话里使用——stop 要读本次对话来归档。
```
