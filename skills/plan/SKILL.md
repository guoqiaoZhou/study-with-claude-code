---
name: swcc-plan
description: Use when the user wants to prepare or review a new technical topic, build or expand a structured knowledge tree / study roadmap for a subject, or update an existing topic's tree. 触发例:「复习 X」「准备面试 X」「给我搭个 X 知识树」「plan X」「更新 X 的知识树」「把这个专题补全点」。
argument-hint: "[topic] [level] [--update|--regenerate]"
user-invocable: true
---

# swcc · plan — 生成 / 更新专题知识树

为一个技术专题搭建（或增量更新）**roadmap 级**的知识树与复习材料。质量标杆是「阶段 → 主题（带核心问题）→ 多条带答案的实质知识点 + 对应资料 + 面试考点」那种密度，而不是一串光秃秃的节点名。

> 动手前先读三份契约:
> - `${CLAUDE_PLUGIN_ROOT}/skills/_shared/data-contract.md` —— 目录布局、节点路径 key、各文件格式、参考资料读取策略、**合并/更新语义**。
> - `${CLAUDE_PLUGIN_ROOT}/skills/_shared/review-rubric.md` —— 强制评审的维度、各 level 深度校准、缺口格式。
> 所有数据写到 `$HOME/.study-with-cc/`。

参数:`$ARGUMENTS` —— 第一个词是 topic（必填），可选 level（`p6`/`p7`/`p8`/`p9`，默认 `p7`），可选 `--update` / `--regenerate`。

---

## 核心原则（贯穿全程，在门控处会复述）

1. **先自己想透，再看书。** 永远**先用自身领域知识构建完整大纲**，再用参考资料锚定/补充。**绝不**因为挂了书就把范围缩到书的目录——书是锚点，不是天花板。
2. **深度优先于条目数。** 每个叶子节点都要能回答「是什么 + 为什么这样设计 + 取舍/边界」，不是列名词。宁可少而透，不要多而空。
3. **质量由流程保证，不靠用户判断。** 生成后**必须**派评审子智能体查漏补缺并据反馈补全——这是强制环节，不问用户「够不够」。
4. **结构可演进，进度不可丢。** 更新已存在专题时，默认增量合并，用户已学的进度一律保留。
5. **不编来源。** 没挂资料就老实靠自身理解，不要假称「依据某书」。
6. **大文件分块写,永不一次性整文件输出。** knowledge-system.md 等大产物按 data-contract 第十二节分块追加 + TodoWrite 跟踪,否则会超时。

---

## 执行流程

| 阶段 | 名称 | 目的 |
|---|---|---|
| 1 | 解析参数 | 取 topic / level / 模式标记 |
| 2 | 存在性检查 & 路由 | 新建 or 更新（增量合并 / 重建） |
| 3 | 询问参考资料 | 收集 PDF/文档/目录的绝对路径 |
| 4 | **独立构建专家大纲** | 先不看书，凭领域知识列全大纲 |
| 5 | **挂载资料并锚定补充** | 用书映射/补充/纠错，标注来源 |
| 6 | 生成 knowledge-system.md | 每个叶子节点写成 roadmap 级 |
| 7 | **强制 subagent 评审 + 补全** | 并行评审 → 合并缺口 → 补全 → 循环 |
| 8 | 写 5 文件 + 自检 | 落盘并 `ls` 验证缺一不可 |
| 9 | 输出摘要 | 含评审补了哪些缺口 |

---

### 阶段 1：解析参数

- 从 `$ARGUMENTS` 取 topic（没给则问用户「要复习/准备哪个专题？」）、level（默认 `p7`）、模式标记。
- 由 topic 生成小写 kebab-case `slug`（转小写、空格转连字符；如「Foo Bar」→ `foo-bar`）。

### 阶段 2：存在性检查 & 路由

看 `$HOME/.study-with-cc/topics/<slug>/` 是否存在:

| 情况 | 路由 |
|---|---|
| 不存在 | **新建模式**:走阶段 3→9 全流程。 |
| 已存在，且无 `--regenerate` | **增量合并模式（默认）**:仍走阶段 4–7 重新深想+评审，但落盘时按 data-contract 第十一节**只增不毁**——新增缺失节点、加深已有节点内容，progress 进度全保留，frontmatter `version` 递增。先读出现有 tree/system/progress 作为基线。 |
| 已存在，且有 `--regenerate` | **重建模式**:全量重写 tree/system，但按节点路径 key **迁移旧 progress**，匹配不上的旧节点在摘要里列出。仅在用户明确要求时用。 |

> 提醒（原则 4）:合并/重建都以「用户已学的进度不丢」为底线。

### 阶段 3：询问参考资料

问用户:**是否有 PDF 书籍、文档、或资料目录作为本专题依据？**（举例:`/Users/you/books/<教材文件>.pdf`）
- **有** → 收集每份的绝对路径、类型、标题，准备写入 `references.json`。
- **无** → `references` 写 `{"references": []}`，纯靠自身理解（阶段 5 跳过书锚定）。
- 更新模式下:沿用已有 `references.json`，并问是否新增。

### 阶段 4：独立构建专家大纲（先不看书）

**这是深度的源头，先做这步、且不要被书牵着走。**

凭你自身领域知识，把专题分解为:**阶段（`##` 分组）→ 主题 → 知识点（叶子节点）**。
- 按 `level`（见 review-rubric 第三节深度校准）确定粒度与深度:`p7` 要覆盖高频面试考点 + 实战调优，密度满足 review-rubric「深度标准」。
- 叶子节点是「可单独考核的知识点」，不是大而空的名词（见 data-contract 第三节节点粒度）。
- 此刻先在心里/草稿列出完整骨架，**不要因为「书里没有」就删点**。

> 提醒（原则 1）:即使挂了书，这一步也**先于**读书。书用来补，不用来框。

### 阶段 5：挂载资料并锚定补充

仅当阶段 3 挂了资料:按 data-contract 第八节读取（PDF 用 Read **分页读目录/前 10 页**提取章节结构；目录列结构挑相关文件；docx 走 best-effort 转换）。然后:
- 把书的章节**映射**到阶段 4 的大纲上，给相关节点标「来源 - 书:第 N 章」。
- 用书**补充细节、纠正错误、补上你漏掉的点**。
- 大纲里书没覆盖、但该 level 该掌握的点，标「来源 - 网络补充」。

> 门控:离开本阶段前确认——大纲是否仍**超出**书的目录范围（应当超出）？若发现自己被书缩窄了，回阶段 4 补回来。

### 阶段 6：生成 knowledge-system.md（**分块写，禁止一次性整文件输出**）

⚠️ 一次性输出几十个 roadmap 级节点会**超时失败**。严格按 data-contract 第十二节「大文件分块写入策略」:

1. 先用 Write 建文件:表头 `# <topic> 知识体系` + 末尾哨兵 `<!-- @end -->`。
2. 用 **TodoWrite** 为每个 `##` 子主题建一个 todo（「生成+落盘 <子主题>」）。
3. **逐子主题**:生成该子主题下所有叶子节点的四段内容（**核心问题 / 核心概念 / 面试考察点 / 来源**，严格按 data-contract 第四节 roadmap 级格式，核心概念每条带答案与原理/取舍/边界、不许只写名词）→ 用 **Edit 替换哨兵**追加该段 → 勾掉该 todo。**一个子主题一次 Edit**，绝不攒着一次写完。
4. 子主题很多时,可按 data-contract 第十二节第 5 步**并行派起草子智能体**(每个只起草一个子主题、只返回文本、不写文件),主体再逐块 Edit 追加。

> 门控:每块落盘后再写下一块。中途超时也能凭 TodoWrite + 已落盘内容续写，不必从头再来。

### 阶段 7：强制 subagent 评审 + 补全（不问用户）

**这是质量闸门，必做。** 用 **Agent 工具并行派发评审子智能体**（一条消息里多个 Agent 调用并发），按 review-rubric 第二节，每维度一个:
- **覆盖度**（始终）、**深度**（始终）、**书本忠实度**（挂了书才派）。

把当前的知识树大纲 + knowledge-system 内容 + topic + level（+ 挂载资料清单）作为上下文交给每个子智能体，要求其**只读、按 review-rubric 第四节的 `gaps[]` JSON 结构返回缺口**。

拿到结果后（原则 3）:
1. 合并三维 `gaps[]`，按 `node+suggestion` 去重。
2. 优先 `high`、再 `medium` 补全 tree（新增节点）与 system（加深内容；**补 system 同样用分块 Edit 追加，见 data-contract 第十二节**）。
3. **循环复审**:补完再派一轮，直到**连续一轮无新增 high/medium**（loop-until-dry），**最多 2–3 轮**。
4. 记下「评审补了哪些缺口」用于摘要。

> 若三个维度的子智能体都返回空缺口,先对照 review-rubric「深度标准」逐节点核对是否真的已达标;若仍不确定,换一个更严格的视角（例如「一个挑剔的资深面试官会追问什么」）再评审一轮。

### 阶段 8：写文件 —— 必须生成 5 样，缺一不可

⚠️ 必须生成下列全部 5 个文件，缺一不可。`config.json` 位于全局根目录（不在 topic 目录内）、`references.json` 即使没有资料也要建为空——这两处位置/条件特殊，**最先创建**。一律 `mkdir -p` 确保目录存在。更新模式下按 data-contract 第十一节合并语义写入，**不得覆盖已有进度**。

| # | 文件 | 位置 | 要点 |
|---|------|------|------|
| 1 | `config.json` | `$HOME/.study-with-cc/`（全局根） | 已存在:读出 → 追加/确认本 topic 在 `topics[]` → `activeTopic` 设为本 slug → 写回（别覆盖别的 topic）。不存在:按 data-contract 第九节新建。 |
| 2 | `references.json` | topic 目录 | 阶段 3 的资料清单;无资料写 `{"references": []}`。**即使为空也必须创建。** |
| 3 | `knowledge-tree.md` | topic 目录 | frontmatter（topic/slug/version/level/**kind**/createdAt，createdAt 用 `date +%F`；`kind` 按专题性质判定:编程类→`coding`，其余→`conceptual`）+ 嵌套 checkbox。新建全 `- [ ]`;更新模式按合并语义只增不毁、`version` 递增。 |
| 4 | `knowledge-system.md` | topic 目录 | 每个叶子节点四段 roadmap 级内容。 |
| 5 | `progress.json` | topic 目录 | 新建:每个**叶子节点**一条 `not_started`/`mastery:0`，`currentNode`=第一个叶子，weakPoints 空，stats 全 0。更新:按合并语义只补新节点、保留旧进度。节点 key 必须与 tree 层级完全一致。 |

### 阶段 8.5：完成前自检（必做）

```
ls -1 "$HOME/.study-with-cc/config.json" \
      "$HOME/.study-with-cc/topics/<slug>/references.json" \
      "$HOME/.study-with-cc/topics/<slug>/knowledge-tree.md" \
      "$HOME/.study-with-cc/topics/<slug>/knowledge-system.md" \
      "$HOME/.study-with-cc/topics/<slug>/progress.json"
```
再确认 `config.json` 的 `activeTopic` 指向本 slug、`topics[]` 含本条目。任一缺失/不符立即补写。

### 阶段 9：输出摘要

```
✅ 知识树已<生成 / 更新>：<topic>（level: <level>）
📊 节点数：<叶子总数>（其中本次新增 <k>）　子主题：<N>
🔍 评审补全：<高优 a 个 / 中优 b 个缺口>（无则「评审通过，无缺口」）
📚 已挂载参考资料：<M> 份
📄 详细知识体系：knowledge-system.md
📁 位置：~/.study-with-cc/topics/<slug>/

下一步：用 /swcc-go 开始复习。
```

---

## 质量基准（达到才算完成）

- 每个叶子节点在 knowledge-system.md 里都有「核心问题 + 多条带答案的核心概念 + 面试考点 + 来源」，满足 review-rubric「深度标准」那五条。
- 知识树的范围**不止于**挂载资料的目录（除非该专题确实就这么大）。
- 评审子智能体确实跑过，且 high/medium 缺口已补或已说明为何不补。
- 5 个文件齐全，更新模式下旧进度零丢失。
