# Study Planner — 输出契约（STUDY_PLAN_OUTPUT）

> **版本**：`mvp-1`  
> **触发**：今日学习计划 / 学习规划类意图  
> **消费者**：用户执行、Review Coach（`study_plan_items`）、可选 `memory/data/daily_study_plan.json`

---

## 1. 交接块格式

````markdown
<!-- STUDY_PLAN_HANDOFF:BEGIN -->
```json
{ ... }
```
<!-- STUDY_PLAN_HANDOFF:END -->
````

---

## 2. 顶层结构

```json
{
  "module": "study_planner",
  "version": "mvp-1",
  "trigger": "daily_plan",
  "user_id": "default_user",
  "plan_date": "2026-07-31",
  "generated_at": "2026-07-31T09:00:00+08:00",
  "data_quality": "partial",
  "data_sources": {
    "mistake_count": 3,
    "confirmed_wrong_count": 0,
    "weak_point_count": 0,
    "has_retrospective_snapshot": true,
    "exam_date": null,
    "days_to_exam": null
  },
  "daily_plan": { },
  "study_plan": { },
  "study_plan_items": [],
  "horizon_preview": []
}
```

---

## 3. `daily_plan`（用户六要素 — 必填）

与产品需求一一对应：

| 字段 | 类型 | 说明 |
|------|------|------|
| `focus_knowledge_domains` | object[] | **今日重点知识领域**；见 §3.1 |
| `recommended_review_question_count` | integer | **推荐复习题数量**（错题重做 + 同类巩固合计） |
| `weak_point_priorities` | object[] | **薄弱点优先级**；见 §3.2 |
| `recommended_materials` | object[] | **推荐学习材料**；见 §3.3 |
| `estimated_study_minutes` | integer | **预计学习时间**（分钟） |
| `completion_status` | enum | **完成状态**：`pending` / `in_progress` / `completed` / `partially_completed` / `skipped` |

### 3.1 `focus_knowledge_domains[]`

```json
{
  "rank": 1,
  "knowledge_domain": "实施整体变更控制",
      "exam_domain": "process",
  "reason": "收藏待巩固 + 复盘知识域排名第 1",
  "allocated_minutes": 25,
  "suggested_actions": ["review_mistake", "knowledge_learning"]
}
```

### 3.2 `weak_point_priorities[]`

```json
{
  "rank": 1,
  "weak_point_id": null,
  "knowledge_domain": "价值交付",
  "priority_score": 72,
  "dominant_error_type": null,
  "source": "mistake_memory_aggregate"
}
```

`weak_point_id` 来自 `weak_points.json` 时有值；否则为 `null`，`source` 标明由错题聚合推算。

### 3.3 `recommended_materials[]`

```json
{
  "material_type": "knowledge_doc",
  "title": "PMP 决策树 — First / Next",
  "path": "knowledge/decision_framework/decision_tree.md",
  "reason": "First 类争议题暴露场景判断薄弱",
  "estimated_minutes": 15
}
```

| `material_type` | 说明 |
|-----------------|------|
| `knowledge_doc` | `knowledge/**` Markdown |
| `mistake_review` | 指定 `mistake_ids` 重做 |
| `drill` | 薄弱点专项刷题（数量在 `target_count`） |
| `video_learning` | 占位；`lesson_ids` P1 |

---

## 4. `study_plan` + `study_plan_items`（对齐 `learning_schema.md`）

### 4.1 `study_plan`

```json
{
  "study_plan_id": "plan_daily_20260731_001",
  "title": "每日学习计划 2026-07-31",
  "start_date": "2026-07-31",
  "end_date": "2026-07-31",
  "status": "active",
  "generated_by": "agent",
  "context_snapshot": {
    "study_stage": "practice",
    "days_to_exam": null,
    "available_minutes": 90
  }
}
```

### 4.2 `study_plan_items[]`

每条为可执行任务，供 Review / Training Coach 消费：

| 字段 | 必填 | 说明 |
|------|------|------|
| `item_id` | 是 | UUID 或 `item_{date}_{seq}` |
| `scheduled_date` | 是 | = `plan_date` |
| `item_type` | 是 | `review_mistake` / `weak_point_drill` / `knowledge_learning` / `mock_exam` |
| `title` | 是 | 任务标题 |
| `target_count` | 否 | 题数或分钟 |
| `target_refs` | 否 | `mistake_ids`、`weak_point_ids`、`knowledge_refs` |
| `priority` | 否 | 同日内排序，越大越先 |
| `status` | 是 | 初始 `pending` |

---

## 5. 用户可见 Markdown 模板（固定顺序）

```markdown
# 今日学习计划

**日期**：{plan_date}  
**预计总时长**：{estimated_study_minutes} 分钟  
**完成状态**：{completion_status 中文}

## 1. 今日重点知识领域

{focus_knowledge_domains 列表：领域 + 理由 + 建议动作}

## 2. 推荐复习题量

- 错题重做：**{n1}** 道
- 同类巩固练习：**{n2}** 道（可选）
- **合计建议**：**{recommended_review_question_count}** 道

## 3. 薄弱点优先级

{weak_point_priorities 有序列表}

## 4. 推荐学习材料

{recommended_materials 列表，含路径或任务说明}

## 5. 今日任务清单

| 优先级 | 任务 | 类型 | 题量/时长 | 状态 |
|--------|------|------|-----------|------|
| ... | ... | ... | ... | pending |

## 6. 执行提示

- 先完成 **review_mistake** 任务 → 可说「开始复习」交给复习流程
- 完成后回复「今日计划完成」或勾选任务，便于下次更新 `completion_status`
```

---

## 6. `data_quality`

| 值 | 含义 |
|----|------|
| `good` | 有 confirmed_wrong + weak_points 或完整复盘快照 |
| `partial` | 主要为收藏/待巩固，或 weak_points 为空但可聚合 |
| `insufficient` | 几乎无个人数据，输出模板化通用计划 |

---

## 7. `horizon_preview`（可选）

MVP 未来 3 天方向（非详细排期）：

```json
{
  "date": "2026-08-01",
  "theme": "敏捷价值与待办优先级巩固",
  "estimated_minutes": 90
}
```

---

## 8. 完成状态更新约定

| 事件 | `completion_status` |
|------|---------------------|
| 刚生成 | `pending` |
| 用户开始执行任一任务 | `in_progress` |
| 全部 `study_plan_items` 完成 | `completed` |
| 部分完成 | `partially_completed` |
| 用户跳过 | `skipped` |

MVP 由 Agent 根据用户对话更新 handoff 中的字段；持久化见 `memory/data/daily_study_plan.json`（可选）。
