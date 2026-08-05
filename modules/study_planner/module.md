# Study Planner — MVP 模块实现

> **所属产品**：PMP AI Coach（单一主 Skill）  
> **模块类型**：专业能力模块 | **非独立 Skill**  
> **版本**：`mvp-1`  
> **数据层**：只读 `memory/data/*`；可选写入当日计划快照

---

## 1. 模块定位

Study Planner 优先读取：

1. **Mistake Memory**（真实错题）  
2. **WeakPoint** / Review Coach 输出  
3. **Learning State**  

**Bookmark Memory** 仅作辅助（如考前「复习收藏重点题」），**不得**把收藏计入错误次数。

```
用户：「今天学什么？」/「帮我定今日计划」
        │
        ▼
读取 memory/data/
  ├── mistake_memory.json
  ├── weak_points.json
  ├── learning_state.json
  └── （可选）review_retrospective.json、memory/user_profile.json
        │
        ▼
规划算法（planning_rules.md）
        │
        ▼
固定 Markdown 日计划 + STUDY_PLAN_OUTPUT
        │
        ▼
可选：memory/data/daily_study_plan.json（最近一次当日计划）
```

**不负责**：单题讲解（→ Question Coach）、错题入库（→ Mistake Coach）、复习过程引导（→ Review Coach）。

**协作**：计划中 `item_type=review_mistake` / `weak_point_drill` 的任务，用户开始执行时由 **Review Coach** 消费。

---

## 2. 触发条件

| 用户输入 | 说明 |
|----------|------|
| 今天学什么 / 今日任务 / 今日学习计划 | 生成 **单日** 计划（默认今天） |
| 学习计划 / 帮我安排今天 / 考前怎么学 | 同上；可含用户约束（「只有 1 小时」） |
| 制定 N 天计划 / 考前 30 天 | MVP：生成 **今日** + 简要 **未来 3 天方向**（`horizon_preview`），完整多周排期为 P1 |

**路由**：`skill.md` §2 P3；**不得**与 P0 题目分析冲突。

---

## 3. 输入契约

### 3.1 必读文件

| 文件 | 用途 |
|------|------|
| `memory/data/mistake_memory.json` | 待复习错题、`repeated_count`、`error_type`、`review_status`（读兼容 `mistake_type`） |
| `memory/data/weak_points.json` | Review / Mistake Coach 聚合的薄弱点与 `priority_score` |
| `memory/data/learning_state.json` | `current_study_stage`、`error_patterns`、`stats`、`sessions` |

### 3.2 可选增强

| 文件 | 用途 |
|------|------|
| `memory/data/review_retrospective.json` | 最近一次 `knowledge_domain_ranking`、`next_training` |
| `memory/user_profile.json` | `exam_date`、`daily_study_time`、`study_stage` |
| `modules/review_coach/aggregation_rules.md` | 与复盘一致的知识域权重（无复盘快照时 Agent 可重算） |

### 3.3 输入 JSON（`PLANNER_INPUT`）

```json
{
  "trigger": "daily_plan",
  "user_utterance": "今天学什么",
  "user_id": "default_user",
  "plan_date": "2026-07-31",
  "options": {
    "available_minutes": null,
    "focus_domain": null,
    "include_mock_exam": false
  }
}
```

| 字段 | 默认 | 说明 |
|------|------|------|
| `plan_date` | 消息日期 ISO 日期 | 计划执行日 |
| `available_minutes` | `user_profile.daily_study_time` 或 `90` | 用户可用学习时长 |
| `focus_domain` | `null` | 用户指定主攻知识域时覆盖自动排序 Top1 |
| `include_mock_exam` | `false` | 距考试 ≤14 天且周末可建议模考块（P1 简提示） |

---

## 4. 执行流程

```
1. Load Memory          ← 三文件 + 可选 profile / retrospective
2. Normalize Records    ← 真实错题 P0 过滤（对齐 mistake_coach/decision_rules.md §0）
3. Score Priorities     ← planning_rules.md §2–§5
4. Allocate Time        ← planning_rules.md §6
5. Build StudyPlanItems ← 对齐 database/learning_schema.md §4
6. Render Daily Plan    ← output_contract.md §3 用户可见结构
7. Emit Handoff         ← STUDY_PLAN_OUTPUT
8. Optional Persist     ← daily_study_plan.json
```

详细步骤见 `workflows/study_plan.md`。

---

## 5. 输出契约

用户可见：**每日学习计划**（六要素）+ 可勾选任务列表。

机器可读：`STUDY_PLAN_HANDOFF` JSON，见 `output_contract.md`。

示例见 `examples/sample_study_plan.json`。

---

## 6. 数据不足时的行为

| 条件 | 行为 |
|------|------|
| 三文件均空或仅模板 | 输出 **通用备考日计划**（ECO 默认权重 + 刷题/阅读模板），`data_quality=insufficient` |
| 无 confirmed_wrong，仅有收藏/争议 | 以 `bookmarked` + `review_retrospective` 知识域排序排任务，`data_quality=partial` |
| 无 `exam_date` | 使用默认节奏（`study_stage` + 固定 90 分钟），标注【待确认考试日期】 |

不得编造用户未产生的错题统计。

---

## 7. Agent 约束

| 约束 | 说明 |
|------|------|
| 只读错题库 | 不修改 `mistake_memory`；完成状态由用户确认或 Review Coach 回写后，Planner 下次读取 |
| 不讲解题目 | 只分配 `mistake_ids` 与材料路径 |
| 枚举一致 | `item_type`、`error_type` 与 `database/learning_schema.md`、`mistake_schema.md` 对齐 |
| 用户无感 | 回复中不出现「Study Planner 模块」 |

---

## 8. 文件索引

| 文件 | 说明 |
|------|------|
| `module.md` | 本文件 |
| `output_contract.md` | `STUDY_PLAN_OUTPUT` 字段与 Markdown 模板 |
| `planning_rules.md` | 优先级、题量、时长、材料推荐算法 |
| `examples/sample_study_plan.json` | 完整示例 |
| `workflows/study_plan.md` | 执行工作流 |
| `database/learning_schema.md` | StudyPlan / StudyPlanItem 权威 Schema |
