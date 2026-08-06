# Learning Schema

> 学习规划、复习执行、薄弱点与进度时间轴数据模型。  
> **消费模块**：Mistake Coach（WeakPoint）、Study Planner（StudyPlan）、Review Coach（ReviewSession）、Training Coach（LearningProgress）。

---

## 1. Purpose

本 Schema 承载 **Review Coach 与 Study Planner 的分工边界**在数据层的体现，并明确与 **三类用户记忆** 的关系：

### 1.1 三类用户记忆 → 学习层

```
QuestionHistory     做题轨迹 / 正确率统计
        │
Mistake Memory      仅真实错题（自动）
        │
        ▼
   WeakPoint 聚合（Mistake Coach）
        │
        ▼
   Review Coach（复盘 / 错误模式）──只读 Mistake + WeakPoint
        │
        ▼
   Study Planner（每日计划）──优先 Mistake + WeakPoint + LearningState
        │                         └── 辅助：Bookmark（考前收藏复习）
        ▼
   LearningProgress / StudyPlan
```

| 数据源 | Review Coach | Study Planner | LearningProgress |
|--------|:------------:|:-------------:|:----------------:|
| **Mistake Memory** | **主输入** | **优先** | 错题事件 |
| **WeakPoint** | 读 | **优先** | — |
| **Learning State** | 读 | **优先** | 同源 |
| **Bookmark Memory** | **禁止当错误分析** | 辅助（考前收藏） | 收藏事件可选 |
| **Question History** | 不做错因主源 | 可做题量/正确率 | 统计 |

### 1.2 实体归属

| 实体 | 归属能力 | 说明 |
|------|----------|------|
| `WeakPoint` | Mistake Coach 写；Planner/Review 读 | **仅从 Mistake** 聚合，不得用 Bookmark 冒充错误 |
| `StudyPlan` + `StudyPlanItem` | Study Planner 写；Review 读 | **规划**：何时学什么、复习哪些题 |
| `ReviewSession` | Review Coach 写 | **执行**：一次复习会话的过程与结果 |
| `LearningProgress` | 全模块写 | 时间轴：汇总每日/每次学习成效 |

---

## 2. WeakPoint（薄弱点）

从 **`Mistake`（Mistake Memory）** 按 `knowledge_point` / `exam_domain` / `error_type` 聚合。

> **禁止**：将 Bookmark 计入 `error_count`。

| 字段名称 | 类型建议 | 是否必须 | 用途 |
|----------|----------|----------|------|
| `weak_point_id` | UUID | **是** | 主键 |
| `user_id` | UUID | **是** | 外键 → User |
| `knowledge_domain` | VARCHAR(128) | **是** | 领域标识，如「冲突管理」 |
| `exam_domain` | ENUM | 否 | `people` / `process` / `business_environment`（**读兼容** `eco_domain`） |
| `error_count` | INTEGER | **是** | 累计错题数 |
| `repeated_error_count` | INTEGER | 否 | 重复做错次数 |
| `last_error_at` | TIMESTAMPTZ | 否 | 最近错误时间 |
| `error_trend` | ENUM | 推荐 | `increasing` / `stable` / `decreasing` |
| `dominant_error_type` | ENUM | 否 | 最高频 `wrong_type` |
| `priority_score` | DECIMAL | 推荐 | 复习优先级（0–100），Planner/Review 排序用 |
| `suggested_direction` | TEXT / JSON | 推荐 | 建议学习行动 |
| `related_mistake_ids` | JSON | 否 | 关联错题 ID 列表（抽样） |
| `updated_at` | TIMESTAMPTZ | **是** | 聚合计算时间 |

对应 `memory/weak_points.md`。

---

## 3. StudyPlan（学习计划）

Study Planner 产出；描述一个备考周期内的学习安排。

| 字段名称 | 类型建议 | 是否必须 | 用途 |
|----------|----------|----------|------|
| `study_plan_id` | UUID | **是** | 主键 |
| `user_id` | UUID | **是** | 外键 → User |
| `title` | VARCHAR(255) | 推荐 | 如「考前 30 天冲刺」 |
| `start_date` | DATE | **是** | 计划开始日 |
| `end_date` | DATE | 否 | 计划结束日；单日任务可为 null |
| `status` | ENUM | **是** | `draft` / `active` / `completed` / `archived` |
| `generated_by` | ENUM | 推荐 | `agent` / `user` / `system` |
| `context_snapshot` | JSON | 否 | 生成时的 `exam_target_date`、阶段、弱项快照 |
| `created_at` | TIMESTAMPTZ | **是** | |
| `updated_at` | TIMESTAMPTZ | **是** | |

---

## 4. StudyPlanItem（计划任务项）

计划的可执行单元。**Review Coach 消费 `item_type=review_mistake` 与 `weak_point_drill`**。

| 字段名称 | 类型建议 | 是否必须 | 用途 |
|----------|----------|----------|------|
| `item_id` | UUID | **是** | 主键 |
| `study_plan_id` | UUID | **是** | 外键 → StudyPlan |
| `user_id` | UUID | **是** | 冗余外键，便于按用户查询 |
| `scheduled_date` | DATE | **是** | 计划执行日 |
| `item_type` | ENUM | **是** | 见 §4.1 |
| `title` | VARCHAR(255) | **是** | 任务标题 |
| `description` | TEXT | 否 | 详细说明 |
| `target_count` | INTEGER | 否 | 目标数量（题数/页数/分钟） |
| `target_refs` | JSON | 否 | 关联资源，见 §4.2 |
| `priority` | INTEGER | 否 | 同日内排序，越大越优先 |
| `status` | ENUM | **是** | `pending` / `in_progress` / `completed` / `skipped` |
| `completed_at` | TIMESTAMPTZ | 否 | 完成时间 |
| `created_at` | TIMESTAMPTZ | **是** | |
| `updated_at` | TIMESTAMPTZ | **是** | |

### 4.1 item_type 枚举（P0）

| 存储值 | 执行模块 | 说明 |
|--------|----------|------|
| `review_mistake` | **Review Coach** | 错题复习；`target_refs.mistake_ids` 必填 |
| `weak_point_drill` | **Review Coach** + Question Coach | 薄弱点专项；`target_refs.weak_point_ids` 必填 |
| `knowledge_learning` | **Training Coach** | 概念/知识域学习；`target_refs.knowledge_refs` 必填 |
| `mock_exam` | Training Coach（P1） | 模拟考试；`target_refs.exam_set` 可选 |
| `video_learning` | **Training Coach** | 视频/微课学习；`target_refs.lesson_ids` 必填 |

### 4.2 target_refs 示例

```json
{
  "mistake_ids": ["550e8400-e29b-41d4-a716-446655440000"],
  "weak_point_ids": ["wp-conflict-001"],
  "knowledge_refs": ["knowledge/pmbok/integration.md"],
  "lesson_ids": ["lesson-agile-roles-01"],
  "exam_set": "模拟卷第 3 套"
}
```

---

## 5. ReviewSession（复习会话）

Review Coach 执行一次复习时创建；**不替代** `StudyPlanItem`，而是记录执行过程。

| 字段名称 | 类型建议 | 是否必须 | 用途 |
|----------|----------|----------|------|
| `session_id` | UUID | **是** | 主键 |
| `user_id` | UUID | **是** | 外键 → User |
| `study_plan_item_id` | UUID | 否 | 若由计划触发，关联任务项 |
| `started_at` | TIMESTAMPTZ | **是** | 开始时间 |
| `ended_at` | TIMESTAMPTZ | 否 | 结束时间 |
| `items` | JSON | **是** | 复习明细数组，见 §5.1 |
| `summary` | TEXT | 否 | 会话小结 |
| `mistakes_reviewed_count` | INTEGER | 推荐 | 复习题数 |
| `mistakes_mastered_count` | INTEGER | 否 | 本次标记掌握数 |
| `created_at` | TIMESTAMPTZ | **是** | |

### 5.1 items[] 元素结构

```json
{
  "mistake_id": "550e8400-e29b-41d4-a716-446655440000",
  "user_reanswer": "B",
  "is_correct": true,
  "review_status_before": "reviewing",
  "review_status_after": "mastered",
  "notes": "用户确认已掌握冲突管理优先级"
}
```

Review Coach 在会话结束时：
1. 批量更新 `Mistake.review_status` / `review_count`
2. 写入 `LearningProgress`
3. 若关联 `study_plan_item_id`，更新 `StudyPlanItem.status = completed`

---

## 6. LearningProgress（学习进度）

时间轴记录；各模块活动后追加。

| 字段名称 | 类型建议 | 是否必须 | 用途 |
|----------|----------|----------|------|
| `progress_id` | UUID | **是** | 主键 |
| `user_id` | UUID | **是** | 外键 → User |
| `study_date` | DATE | **是** | 学习日期 |
| `session_type` | ENUM | 推荐 | `daily` / `review` / `practice` / `reading` / `mock_exam` |
| `source_module` | ENUM | 推荐 | `question_coach` / `mistake_coach` / `review_coach` / `study_planner` / `training_coach` |
| `learning_content` | JSON | **是** | 活动内容摘要 |
| `questions_attempted` | INTEGER | 否 | 做题数 |
| `questions_correct` | INTEGER | 否 | 做对数 |
| `accuracy_rate` | DECIMAL | 否 | 正确率 0–1 |
| `mistakes_reviewed` | INTEGER | 否 | 复习错题数 |
| `weak_points_touched` | JSON | 否 | 涉及薄弱域标签 |
| `mastery_status` | TEXT / JSON | 否 | 掌握情况描述 |
| `next_plan` | TEXT / JSON | 否 | Study Planner 写入的下一步建议 |
| `study_duration_minutes` | INTEGER | 否 | 学习时长 |
| `notes` | TEXT | 否 | 用户笔记 |
| `created_at` | TIMESTAMPTZ | **是** | |

对应 `memory/learning_progress.md`。

### 6.1 learning_content 示例

```json
[
  { "type": "review", "mistake_ids": ["..."], "session_id": "..." },
  { "type": "reading", "topic": "风险管理", "ref": "knowledge/pmbok/risk.md" },
  { "type": "practice", "topic": "变更管理", "count": 20 }
]
```

---

## 7. Review Coach vs Study Planner 数据分工

| 动作 | 写入实体 | 负责模块 |
|------|----------|----------|
| 生成两周计划 | `StudyPlan`, `StudyPlanItem[]` | Study Planner |
| 输出「今日 3 条任务」 | `StudyPlanItem`（`scheduled_date=today`） | Study Planner |
| 用户开始复习错题 | `ReviewSession` | Review Coach |
| 更新错题掌握度 | `Mistake.review_status` | Review Coach |
| 标记计划任务完成 | `StudyPlanItem.status` | Review Coach（执行后回写） |
| 记录今日学了什么 | `LearningProgress` | 执行方模块 |
| 生成明日建议 | `LearningProgress.next_plan` | Study Planner |

---

## 8. 最小 JSON 示例（StudyPlan + ReviewSession）

```json
{
  "study_plan": {
    "study_plan_id": "plan-001",
    "user_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
    "title": "本周错题攻坚",
    "start_date": "2026-07-29",
    "end_date": "2026-08-04",
    "status": "active",
    "items": [
      {
        "item_id": "item-001",
        "scheduled_date": "2026-07-29",
        "item_type": "review_mistake",
        "title": "复习冲突管理错题 3 道",
        "target_refs": { "mistake_ids": ["m1", "m2", "m3"] },
        "status": "pending"
      },
      {
        "item_id": "item-002",
        "scheduled_date": "2026-07-29",
        "item_type": "knowledge_learning",
        "title": "阅读：管理团队过程",
        "target_refs": { "knowledge_refs": ["knowledge/pmbok/team.md"] },
        "status": "pending"
      }
    ]
  },
  "review_session": {
    "session_id": "rs-001",
    "user_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
    "study_plan_item_id": "item-001",
    "started_at": "2026-07-29T20:00:00+08:00",
    "ended_at": "2026-07-29T20:25:00+08:00",
    "items": [
      {
        "mistake_id": "m1",
        "user_reanswer": "B",
        "is_correct": true,
        "review_status_before": "reviewing",
        "review_status_after": "mastered"
      }
    ],
    "mistakes_reviewed_count": 3,
    "mistakes_mastered_count": 1
  }
}
```
