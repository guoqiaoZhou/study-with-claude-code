---
name: swcc-mock
description: Use when the user wants a mock interview / timed simulated exam over a topic, not a guided study session. 触发:「模拟面试」「mock」「面我一次」「来场模拟」「模拟考」「mock interview」「考一套」。
argument-hint: "[topic] [level]"
user-invocable: true
---

# swcc · mock — 模拟面试

跨节点连续出题 + 综合评分报告。一气呵成地考一整套,最后给分数、亮点、改进建议。

> 开始前先读:
> - `${CLAUDE_PLUGIN_ROOT}/skills/_shared/data-contract.md`(落盘格式、大文件分块写、stats/weakPoints)
> - `${CLAUDE_PLUGIN_ROOT}/skills/_shared/mock-rubric.md`(题型配比、评分、回写)
> 本技能会**写**:新增一份 mock-sessions 报告,并回写 progress.json 的 weakPoints 与 stats。

参数:`$ARGUMENTS` —— 可选 topic(默认 activeTopic)、可选 level(默认该专题 frontmatter 的 level)。

---

## 核心原则

1. **先连续作答,再统一评分。** 像真面试:一题接一题问完,中途只简短承接、不长篇点评;全部答完后一次性出报告。
2. **评分由独立子智能体做。** 不靠主对话主观印象;按 mock-rubric 客观打分。
3. **抽样,不重定节点掌握度。** 一场面试覆盖有限,只回写 weakPoints + 计一次 stats,**不直接改各节点 mastery**。
4. **题型随专题 kind。** 算法/实操题**仅 `kind: coding`** 出;`conceptual` 专题用综合应用题替代。
5. **建议用时,不真倒计时。** 每题标「建议 X 分钟」即可。

---

## 流程

### 1. 加载 & 选题篮
- topic 缺省 → activeTopic(缺失按数据契约第九节兜底)。读 `knowledge-tree.md`(取 `level`、`kind`)、`knowledge-system.md`、`progress.json`、`references.json`。
- 读全局 `learner-profile.md`(若存在),据其中风格偏好**调整出题/点评风格**——只调风格,不放松题型配比与独立评分纪律。见 data-contract 第十四节。
- 按 mock-rubric 第一节组卷:**偏向 weakPoints 与低 mastery 节点(查漏)+ 主干高频点(覆盖)**;题型配比按 `kind`;难度按 `level`。

宣布:
```
🎤 模拟面试：<topic>（level <level>，kind <kind>）
题目：概念 ×3 + 场景 ×2<+ 算法/实操 ×1 若 coding>　建议总时长 ~<n> 分钟
开始后我会一题题问，全部答完再统一给报告。
```

### 2. 连续作答
- **一题一题问**,每题标注题型 + 建议用时。用户答完只做**简短承接**(「收到,下一题」),不展开点评——把点评留到报告。
- 用户中途要求看进度/跳过 → 允许跳过(记 0 分),不强制。

### 3. 派评估子智能体评分
全部答完后,用 **Agent 工具**把整场题目与作答 + mock-rubric 交给评估子智能体,要求其**只评估、不写文件**,按 mock-rubric 第五节返回 JSON(总分、分题型、逐题、weakConcepts、advice)。

### 4. 写 mock 报告（分块）
按数据契约第十二节,在 `mock-sessions/<时间戳>.md`(时间戳 `date +%Y-%m-%d-%H-%M-%S`)分块写:先骨架,再逐题型 Edit 追加。结构:
```markdown
# 模拟面试报告：<YYYY-MM-DD HH:mm>
## 总分：<totalScore>/100
## 概念题 <byType.concept>/30
- Q1 <题干>（<score>/10）<亮点/欠缺>
...
## 场景题 <byType.scenario>/40
...
## 算法/实操题 <byType.algo>/30   （conceptual 专题省略）
...
## 改进建议
1. <advice>
```

### 5. 回写 progress.json
- **weakConcepts → weakPoints**:逐条追加(node/concept/mastery/createdAt=今天/nextReview 按艾宾浩斯 reviewCount=1/reviewCount=0/consecutivePass=0;已存在同概念则更新 mastery、不重复加)。
- `stats`:`totalReviewCount += 1`、`totalReviewTime` 加本场估时、全局 `streak`、`lastReviewDate`=今天。
- **不改各节点 mastery / status**。

### 6. 输出摘要
```
🎤 模拟面试报告　总分 <totalScore>/100
　概念 <c>/30 ｜ 场景 <s>/40 ｜ 算法 <a>/30
🔴 新增/加重薄弱点：<列表>
💡 改进建议：<前 2 条>
📁 报告：~/.study-with-cc/topics/<slug>/mock-sessions/<时间戳>.md
```

---

## 质量基准
- 题型配比与 `kind` 一致(conceptual 不出纯算法题);选题体现了"查漏(薄弱/低分)+覆盖(主干)"。
- 评分来自独立子智能体;报告分块写、未超时。
- 只回写了 weakPoints + stats,未改各节点 mastery。
