---
name: swcc-deep
description: Use when the user wants to deeply explore or broaden a single concept — cross-domain connections, analogies, comparisons, extension questions — not test it. 触发:「深挖 X」「拓展一下 X」「deep X」「X 还能怎么理解」「X 和别的领域有什么联系」「横向对比 X」。
argument-hint: "[topic] [concept]"
user-invocable: true
---

# swcc · deep — 概念深挖拓展

把一个概念**横向拓宽**:跨领域连接、类比、技术对比、延伸问题。这是"拓宽 + 沉淀",不是测试,不影响掌握度。

> 开始前先读数据契约:`${CLAUDE_PLUGIN_ROOT}/skills/_shared/data-contract.md`(参考资料读取、deep-notes 目录)。本技能只**新增** `deep-notes/` 笔记,不改 progress/tree。

参数:`$ARGUMENTS` —— topic(默认 activeTopic)+ **concept(要深挖的概念,必填)**。没给 concept 就问用户要挖哪个。

---

## 核心原则

1. **拓宽,不考核。** 目的是打开视野、建立联系,不打分、不记薄弱点、不动 progress。
2. **可选联网,但离线可用是底线。** 默认尝试用 `WebSearch`/`WebFetch` 拉跨领域/最新材料;**联网失败就回落到自身知识 + 已挂资料**,绝不因断网报错。
3. **沉淀成笔记。** 结果写入 `deep-notes/<concept-slug>.md`,让深挖积累成知识资产。
4. **基于已有锚点。** 先看该概念在 knowledge-system.md 里的定位,再向外扩展,别脱离主题乱发散。

---

## 流程

### 1. 定位概念
- 在 `knowledge-tree.md` / `knowledge-system.md` 里找到该 concept 所属节点与已有内容(挂了资料按数据契约第八节读相关章节)。找不到精确匹配 → 跟用户确认要挖的是哪个。
- 读全局 `learner-profile.md`(若存在),据其中讲解偏好**调整深挖的讲法/类比风格**——只调风格,不放松"四段都要有实质内容"的要求。见 data-contract 第十四节。

### 2. （可选）联网补料
- 尝试 `WebSearch`/`WebFetch` 找该概念在其他领域的应用、最新进展、经典对比。
- **失败/不可用** → 跳过,用自身知识 + 已挂资料继续。在笔记里标注是否用了联网。

### 3. 生成深挖内容（四段）
- **🌐 跨领域连接**:这个概念/机制在别的技术领域(或别学科)里有什么对应、相似或迁移?
- **🏠 类比**:用一个直观的现实类比解释它的核心思想。
- **🔄 技术对比**:与同类替代方案/相邻概念横向比较,讲清设计哲学差异。
- **❓ 延伸问题**:抛出几个开放问题,引导进一步思考(可作为以后 mock/go 的难题来源)。

### 4. 沉淀到 deep-notes
- 写 `$HOME/.study-with-cc/topics/<slug>/deep-notes/<concept-slug>.md`(concept-slug = 概念的小写 kebab-case)。已存在则**追加**一段带日期的新内容,不覆盖旧笔记。
```markdown
# 深挖：<concept>
（节点：<节点路径>　<date +%F>　来源：<自身知识 / 联网 / 资料>）

## 🌐 跨领域连接
…
## 🏠 类比
…
## 🔄 技术对比
…
## ❓ 延伸问题
1. …
```

### 5. 输出摘要
```
🔍 已深挖：<topic> / <concept>
　跨领域 <n> 条 ｜ 对比 <m> 条 ｜ 延伸问题 <k> 个　来源：<自身 / 联网>
📁 笔记：~/.study-with-cc/topics/<slug>/deep-notes/<concept-slug>.md
```

---

## 质量基准
- 四段都围绕该概念、有实质内容,不空泛发散。
- 联网不可用时正常产出(用自身知识),并如实标注来源。
- 只写了 deep-notes/ 笔记,progress/tree 未动。
