# 模拟面试评分准则（Mock Rubric）

> 本文件供 `mock` 技能的评估子智能体用:把整场模拟面试的题目与作答交给它,由它按本准则逐题打分、合成总分、产出报告骨架。**评分由独立子智能体完成,不由主对话拍脑袋。**

---

## 一、题型与配比

按 level 选难度,按专题 `kind` 决定是否出实操题:

| 题型 | 数量 | 建议用时 | 适用 |
|---|---|---|---|
| 概念题 | 3 | 5 min/题 | 所有专题 |
| 场景题 | 2 | 10 min/题 | 所有专题 |
| 算法/实操题 | 1 | 20 min/题 | **仅 `kind: coding`**;`conceptual` 专题用一道综合应用题替代或省略 |

**选题来源**:偏向 `weakPoints` 与近期低 mastery 节点(查漏),再补主干高频点(覆盖)。按 `level` 调难度。

---

## 二、单题评分（每题 0–10）

| 分段 | 含义 |
|---|---|
| 0–3 | 答不上来 / 跑题 / 关键错误 |
| 4–6 | 答到结论或部分要点,但原理/取舍/边界讲不清 |
| 7–8 | 原理清楚、能讲取舍、基本完整 |
| 9–10 | 原理 + 取舍 + 边界 + 实例俱全,能反驳替代方案 |

每题记录:得分、亮点、欠缺点(供改进建议)。

---

## 三、合成总分（满分 100）

按题型加权(题数不齐时按实际题型占比归一):
- 概念题合计 30 分(3×10)
- 场景题合计 40 分(2×20)
- 算法/实操题 30 分(1×30);若该专题无此题型,把这 30 分按比例并入概念+场景。

总分 = 各题型得分之和,折算到 100。

---

## 四、回写

mock 据评估结果:
- **答砸的题(单题 ≤ 5)涉及的概念 → 回写 `weakPoints`**(由 stop 同款规则维护;mock 直接追加到 progress.json 的 weakPoints,带 node/concept/mastery/createdAt/nextReview/reviewCount=0/consecutivePass=0)。
- `stats`:计一次复习(totalReviewCount += 1、totalReviewTime 加本场估时、streak/lastReviewDate 按 data-contract)。
- **不直接改各节点 mastery**——一场面试是抽样,不足以重定节点掌握度;只通过 weakPoints 影响后续。

---

## 五、评估子智能体输出（JSON）

```json
{
  "totalScore": 72,
  "byType": { "concept": 20, "scenario": 28, "algo": 14 },
  "items": [
    { "type":"concept", "q":"<题干摘要>", "score":7, "note":"<亮点/欠缺>" }
  ],
  "weakConcepts": [ { "node":"<节点路径>", "concept":"<答砸涉及的概念>", "mastery":4 } ],
  "advice": [ "<改进建议1>", "<改进建议2>" ]
}
```

> mock 主体据此写 mock-sessions 报告、回写 weakPoints + stats。子智能体只评估、不写文件。
