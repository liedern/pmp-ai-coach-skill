# Study Plan Workflow（每日学习计划）

> Study Planner 执行工作流：读取 Memory，按 `planning_rules.md` 生成每日学习计划与 `STUDY_PLAN_OUTPUT`。
>
> **不负责**复习执行（→ Review Coach）与单题讲解（→ Question Coach）。

---

## 0. 执行原则

| 原则 | 说明 |
|------|------|
| 数据驱动 | 题量、薄弱点、时长来自 `memory/data/*`，不编造 |
| 与复盘一致 | 知识域权重优先融合 `review_retrospective.json`（若存在） |
| 真实错题优先 | 复习队列对齐 Mistake Coach P0（`user_answer` ≠ `correct_answer`） |
| MVP 无库 | 可选写 `memory/data/daily_study_plan.json` |

---

## 1. Trigger（触发条件）

| 触发场景 | 识别信号 |
|----------|----------|
| 今日学习 | 「今天学什么」「今日任务」「今日学习计划」 |
| 学习规划 | 「帮我安排今天」「学习计划」「考前怎么学」 |
| 带约束 | 「只有 1 小时」「重点攻风险管理」→ 写入 `PLANNER_INPUT.options` |

**不触发**：完整题目截图 → Question Coach（P0）。

---

## 2. Load Memory

按顺序读取：

```
memory/data/mistake_memory.json
memory/data/weak_points.json
memory/data/learning_state.json
memory/user_profile.json          // 可选
memory/data/review_retrospective.json // 可选
```

校验：

| 检查项 | 失败处理 |
|--------|----------|
| JSON 可解析 | 使用空数组默认值，`data_quality=insufficient` |
| `user_id` 一致 | 默认 `default_user` |

---

## 3. Normalize & Classify Mistakes

对 `mistakes[]` 应用 `modules/study_planner/planning_rules.md` §1：

- 标记 `confirmed_wrong` / `bookmarked` / `disputed` / `excluded`
- P0：`user_answer` 与 `correct_answer` 均非空且不等 → 强制视为复习候选

---

## 4. Score & Prioritize

执行 `planning_rules.md` §2–§5：

1. 计算 `domain_score` → **今日重点知识领域** Top 3  
2. 应用 ECO 权重 → **薄弱点优先级**  
3. 合并 `weak_points.json`（若有）  
4. 读取 `days_to_exam`、`study_stage` → `pace` 与阅读/刷题比例  

---

## 5. Allocate Questions & Time

执行 `planning_rules.md` §7–§8：

- `recommended_review_question_count`  
- `estimated_study_minutes`  
- `recommended_materials[]`  

---

## 6. Build StudyPlanItems

执行 `planning_rules.md` §10：

- 生成 `study_plan` + `study_plan_items[]`（对齐 `database/learning_schema.md`）  
- 组装 `daily_plan` 六要素，`completion_status` 初始为 `pending`  

---

## 7. Fixed Output（用户可见）

按 `modules/study_planner/output_contract.md` §5 Markdown 模板输出。

末尾附：

````markdown
<!-- STUDY_PLAN_HANDOFF:BEGIN -->
```json
{ ... }
```
<!-- STUDY_PLAN_HANDOFF:END -->
````

---

## 8. Optional Persist

写入 `memory/data/daily_study_plan.json`：

```json
{
  "version": "mvp-1",
  "user_id": "default_user",
  "last_daily_plan": { },
  "updated_at": "2026-07-31T09:00:00+08:00"
}
```

用户未要求保存时可跳过；下次规划可读此文件恢复 `completion_status`。

---

## 9. 与其他模块的关系

| 模块 | 关系 |
|------|------|
| Mistake Coach | 提供 `mistake_memory`、真实错题字段 |
| Review Coach | 提供 `weak_points`、`review_retrospective`；**执行** `review_mistake` 任务 |
| Training Coach | **执行** `knowledge_learning` 任务 |
| Question Coach | 不直接调用；drill 阶段用户刷题后仍走 P0 |

---

## 10. 工作流总览

```
用户：今日学什么
        │
        ▼
  ┌─────────────┐
  │ 1. Load     │
  └──────┬──────┘
         ▼
  ┌─────────────┐
  │ 2. Classify │
  └──────┬──────┘
         ▼
  ┌─────────────┐
  │ 3. Score    │
  └──────┬──────┘
         ▼
  ┌─────────────┐
  │ 4. Allocate │
  └──────┬──────┘
         ▼
  ┌─────────────┐
  │ 5. Items    │
  └──────┬──────┘
         ▼
  ┌─────────────┐
  │ 6. Output   │
  └─────────────┘
```
