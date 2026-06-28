---
name: plan
description: 为一个技术专题生成结构化知识树和复习材料（知识树骨架 + 详细知识体系 + 进度初始化），可选地结合用户提供的 PDF/文档/资料目录作为依据。Use this skill when the user wants to start preparing a new topic, build a knowledge tree, plan a study/review roadmap, or says things like "复习 JVM"、"plan JVM"、"准备面试 React"。
argument-hint: "[topic] [level]"
user-invocable: true
---

# in-prep · plan — 生成专题知识树

为一个技术专题从零搭建知识树与复习材料。**一期只做「生成」，不做「迭代/增量更新」**。

> 开始前先读数据契约：`${CLAUDE_PLUGIN_ROOT}/skills/_shared/data-contract.md`，严格按其中的目录布局、节点路径 key、各文件格式、参考资料读取策略来操作。所有数据写到 `$HOME/.in-prep/`。

参数：`$ARGUMENTS` —— 第一个词是 topic（专题名，必填），可选第二个词是 level（`p6`/`p7`/`p8`/`p9`，默认 `p7`）。

## 流程

### 1. 解析参数
- 从 `$ARGUMENTS` 取 topic 和 level。若没给 topic，问用户要复习什么专题。
- 由 topic 生成小写 kebab-case 的 `slug`（如「JVM」→ `jvm`，「Spring Boot」→ `spring-boot`），作为目录名。

### 2. 检查是否已存在
- 看 `$HOME/.in-prep/topics/<slug>/` 是否存在。
- **若已存在**：告知用户「该专题已建过，一期暂不支持迭代更新」，读出现有 `knowledge-tree.md` 展示给用户，然后**停止**（不要覆盖已有数据）。

### 3. 询问参考资料
问用户：是否有 PDF 书籍、文档、或资料目录作为本专题的学习依据？（给个例子：`/Users/you/books/深入理解JVM.pdf`）
- **有** → 收集每份资料的绝对路径、类型、标题，准备写入 `references.json`。然后按数据契约第八节读取资料：
  - PDF：用 Read 读**目录页/前 10 页**提取章节结构，作为知识分解的主要依据。
  - 目录：列出结构，挑相关文件。
  - docx 等：按契约做 best-effort 转换。
- **无** → `references` 为空数组，纯靠你自身理解生成。

### 4. 分解知识树
把专题分解为 **3–5 个子主题**，每个子主题再分解为 **3–5 个核心概念/子节点**（可有更深层级）。
- 按 level 校准深度：p6 基础、p7 进阶、p8/p9 专家（更多冷门点、源码级、调优排查）。
- **若挂了参考资料**：知识树的章节结构与内容**以资料为主、结合你的理解补全**，不要脱离资料另起炉灶。

### 5. 写文件（到 `$HOME/.in-prep/topics/<slug>/`）
按数据契约格式分别生成：
1. `knowledge-tree.md` —— YAML frontmatter（topic/slug/version/level/createdAt，createdAt 用 `date +%F`）+ 嵌套 checkbox，**所有节点初始为 `- [ ]`**。
2. `knowledge-system.md` —— 每个叶子节点的「核心概念」+「面试考察点」。
3. `progress.json` —— 为**每个叶子节点**建一条记录，全部 `status: not_started`、`mastery: 0`；`currentNode` 设为第一个叶子节点；weakPoints 空；stats 全 0。
4. `references.json` —— 第 3 步收集的资料清单（无则 `{"references": []}`）。
5. `config.json`（在 `$HOME/.in-prep/`）—— 若不存在则创建；把本专题追加进 `topics[]` 并把 `activeTopic` 设为本 slug。

> 节点路径 key 必须和 knowledge-tree.md 的层级一致（见数据契约第二节）。创建目录用 `mkdir -p`。

### 6. 输出摘要
```
✅ 知识树已生成：<topic>（level: <level>）
📊 节点数：<叶子节点总数>　子主题：<N>
📄 详细知识体系：knowledge-system.md
📚 已挂载参考资料：<M> 份
🔴 未开始：<叶子节点总数>
📁 位置：~/.in-prep/topics/<slug>/

下一步：用 /go 开始复习。
```
