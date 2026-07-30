# Review Coach — 复盘输出契约（REVIEW_RETROSPECTIVE_OUTPUT）

> **版本**：`mvp-1`  
> **触发**：用户「复盘」  
> **消费者**：用户阅读、Study Planner（P1）、`memory/data/review_retrospective.json`

---

## 1. 交接块格式

````markdown
<!-- REVIEW_RETROSPECTIVE:BEGIN -->
```json
{ ... }
```
<!-- REVIEW_RETROSPECTIVE:END -->
````

---

## 2. 顶层结构

```json
{
  "module": "review_coach",
  "version": "mvp-1",
  "trigger": "retrospective",
  "user_id": "default_user",
  "generated_at": "2026-07-30T22:52:00+08:00",
  "data_quality": "partial",
  "data_sources": {
    "mistake_count": 3,
    "confirmed_wrong_count": 0,
    "bookmarked_count": 2,
    "disputed_count": 1
  },
  "knowledge_domain_ranking": [],
  "mistake_type_analysis": {},
  "weak_points_snapshot": [],
  "learning_suggestions": [],
  "next_training": {},
  "pending_review_mistakes": []
}
```

---

## 3. 字段定义

### 3.1 `data_quality`

| 值 | 含义 |
|----|------|
| `good` | 有 confirmed_wrong + 错因，统计可靠 |
| `partial` | 有记录但错因不足或多为收藏/争议 |
| `insufficient` | 无 `mistakes` 或无可分析字段 |

### 3.2 `knowledge_domain_ranking[]`

见 `aggregation_rules.md` §2.4。

### 3.3 `mistake_type_analysis`

```json
{
  "insufficient_data": false,
  "total_wrong_with_type": 2,
  "by_type": [],
  "top_patterns": [],
  "summary": "一句话总结错误类型分布"
}
```

### 3.4 `weak_points_snapshot[]`

与 `database/learning_schema.md` WeakPoint 字段子集对齐：

`knowledge_domain`, `label`, `priority_score`, `error_trend`, `dominant_mistake_type`, `suggested_direction[]`, `source`（`weak_points.json` | `computed`）

### 3.5 `learning_suggestions[]`

```json
{
  "priority": "high | medium | low",
  "action": "string",
  "ref": "string | null",
  "rationale": "string"
}
```

### 3.6 `next_training`

```json
{
  "primary": {
    "training_type": "review_mistake | weak_point_drill | knowledge_learning | mixed",
    "title": "string",
    "mistake_ids": [],
    "knowledge_domains": [],
    "knowledge_refs": [],
    "target_question_count": null,
    "estimated_minutes": 30
  },
  "alternatives": []
}
```

### 3.7 `pending_review_mistakes[]`

```json
{
  "mistake_id": "string",
  "question_id": "string",
  "review_status": "new",
  "repeated_count": 1,
  "memory_rule": "string",
  "ingestion_tag": "bookmarked"
}
```

---

## 4. 用户可见 Markdown 模板

```markdown
# 学习复盘

> 数据范围：【事实】基于 memory 中 N 条记录（M 条确认错题）

## 1. 高频错误知识领域排序

| 排名 | 知识领域 | 权重分 | 错题数 | 收藏/待巩固 |
| ...

## 2. 高频错误类型分析

...

## 3. 当前薄弱点

...

## 4. 学习建议

1. ...

## 5. 推荐下一步训练方向

**主推**：...
**备选**：...
```

---

## 5. 校验规则

| 规则 | 说明 |
|------|------|
| R1 | `module` = `review_coach` |
| R2 | 所有计数可追溯到 `mistake_memory` |
| R3 | `insufficient_data=true` 时 `by_type` 可为空，但须 `summary` 说明 |
| R4 | 不得虚构 `mistake_id` |
