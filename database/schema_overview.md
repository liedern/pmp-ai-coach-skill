# Database Schema Overview

> PMP AI Coach 共享数据层总览。定义多模块共用的实体、关系与读写边界。  
> **目标**：支撑 Skill MVP（`memory/` 映射）与未来 Web 产品（关系型 DB + API）无缝扩展。

---

## 1. 设计原则

| 原则 | 说明 |
|------|------|
| **单用户中心 → 多用户就绪** | 所有学习数据以 `User` 为根，经 `user_id` 关联 |
| **三类记忆严格分离** | Mistake ≠ Bookmark ≠ QuestionHistory |
| **模块共享写、分责读** | 无模块私有表；通过 Schema 约定读写权限 |
| **枚举稳定** | 对外 **`error_type` / `exam_domain` / `error_reason`**；`review_status`、`result` 使用 snake_case |
| **字段读写** | **写**：仅 canonical 三字段（+ `knowledge_point` 等）；**读**：兼容 `mistake_type` / `eco_domain` / `mistake_reason`；**不迁移**历史 JSON |
| **快照冗余** | Mistake 可保留题干快照；主题库以 Question 为准 |
| **事实与推测分离** | 关键推断配 `confidence_level`；题库标答为 `platform_answer`，判题以 `adjudication_answer` 为准（v0.1.1 AEL） |
| **Memory 映射** | `memory/data/*.json` 为 Agent 运行时视图 |

### 1.1 对外字段规范（canonical）

| 对外（写入 / API / 新 DATA_HANDOFF） | 历史只读别名 |
|--------------------------------------|----------------|
| `error_type` | `mistake_type` |
| `exam_domain` | `eco_domain` |
| `error_reason` | `mistake_reason` |

- **新生成数据禁止写入别名**（含 `mistake_memory.json` 新增/更新条、`QUESTION_OUTPUT`、示例模板）。
- **读取**：`coalesce(error_type, mistake_type)`、`coalesce(exam_domain, eco_domain)`、`coalesce(error_reason, mistake_reason)`。
- **不迁移**已有 `memory/data/*.json` 中的旧键名。

### 1.2 Answer Validation（v0.1.1）

- Handoff / History：`answer_evaluation` 对象（见 `database/question_schema.md` §3.1）。  
- Mistake：`correct_answer` = 当次 `adjudication_answer`；**禁止** `answer_disputed`。  
- 争议：仅 History（`answer_disputed` + `answer_evaluation`）；复盘不计入错因 share（见 Review `aggregation_rules.md`）。

---

## 2. 三类用户记忆（P0）

| 记忆 | 实体 | Memory 文件 | 触发 |
|------|------|-------------|------|
| **Mistake Memory** | `Mistake` | `mistake_memory.json` | 答错自动 |
| **Bookmark Memory** | `Bookmark` | `bookmark_memory.json` | 用户主动收藏 |
| **Question History** | `QuestionHistory` | `question_history.json` | 每次做题自动 |

---

## 3. 实体关系（ER）

```
┌─────────────┐
│    User     │
└──────┬──────┘
       │ 1
       ├──────────────────┬──────────────────┬────────────────┐
       │ N                │ N                │ N              │ N
┌──────▼──────┐   ┌───────▼────────┐  ┌──────▼──────┐  ┌─────▼──────┐
│  Mistake    │   │QuestionHistory │  │  Bookmark   │  │ WeakPoint  │
└──────┬──────┘   └───────┬────────┘  └──────┬──────┘  └─────▲──────┘
       │                  │                  │               │
       │         ┌────────▼──────────────────▼──┐            │ 仅聚合自 Mistake
       └────────►│         Question             │────────────┘
                 └──────────────────────────────┘

┌─────────────────┐     ┌─────────────┐
│ ReviewSession   │     │  StudyPlan  │──► StudyPlanItem
└─────────────────┘     └─────────────┘
┌─────────────────┐
│LearningProgress │
└─────────────────┘
```

---

## 4. Schema 文件索引

| 文件 | 核心实体 | 主要消费模块 |
|------|----------|--------------|
| `user_schema.md` | `User` | 全模块 |
| `question_schema.md` | `Question`, `QuestionHistory`, `Bookmark` | Question Coach；History/Bookmark 分流 |
| `mistake_schema.md` | `Mistake` | Question → Mistake Coach → Review |
| `learning_schema.md` | `WeakPoint`, `StudyPlan`, `StudyPlanItem`, `ReviewSession`, `LearningProgress` | Mistake / Review / Study Planner |

---

## 5. 模块读写矩阵

| 实体 | Question Coach | Mistake Coach | Review Coach | Study Planner | Training Coach |
|------|:--------------:|:-------------:|:------------:|:-------------:|:--------------:|
| `User` | R | R | R | R/W | R |
| `Question` | R/W | R | R | — | R |
| `QuestionHistory` | W | R | R（`result`、争议 `answer_disputed`） | R（正确率） | — |
| `Bookmark` | W* | — | **不读做错误分析** | R（辅助） | — |
| `Mistake` | 触发 W | R/W | R/W | R | — |
| `WeakPoint` | — | R/W | R | R | R |
| `StudyPlan` / Item | — | — | R/W | R/W | R |
| `ReviewSession` | — | — | R/W | R | — |
| `LearningProgress` | — | — | W | R/W | W |

\* Bookmark：用户主动收藏时由 Question Coach / 路由写入。

---

## 6. Memory 层映射

| Memory 文件 | Database 实体 |
|-------------|---------------|
| `memory/user_profile.json` | `User` |
| `memory/data/mistake_memory.json` | `Mistake[]` |
| `memory/data/bookmark_memory.json` | `Bookmark[]` |
| `memory/data/question_history.json` | `QuestionHistory[]` |
| `memory/data/weak_points.json` | `WeakPoint[]` |
| `memory/data/learning_state.json` | Learning 运行时状态 |
| `memory/data/review_retrospective.json` | Review 快照 |
| `memory/data/daily_study_plan.json` | StudyPlan 快照 |

---

## 7. 跨模块数据流（产品闭环）

```
用户做题 / 截图
        │
        ▼
 Question Coach（分析 + DATA_HANDOFF）
        │
        ├──► QuestionHistory（始终）
        │
        ├──► 若 user_answer ≠ correct_answer
        │         │
        │         ▼
        │    Mistake Coach（自动入库，无需「保存错题」）
        │         │
        │         ▼
        │    Mistake Memory → WeakPoint
        │
        └──► 若用户主动收藏
                  │
                  ▼
             Bookmark Memory

Mistake Memory
        │
        ▼
 Review Coach（复盘 / 错误模式）
   - 错因分析：**主要**来自 Mistake Memory
   - History：**补充**答题趋势、重复作答、争议题（`answer_disputed`）
   - Bookmark：**不参与**错误统计
        │
        ▼
 weak_points + learning_state
        │
        ▼
 Study Planner（每日计划）── 优先 Mistake + WeakPoint + LearningState
                              辅助 Bookmark（考前收藏复习）
```

### 7.1 错题自动闭环

```
Question Coach
  └─ 答错 → 自动 handoff Mistake Coach
Mistake Coach
  └─ create/update Mistake + aggregate WeakPoint
Review Coach
  └─ 错因来自 Mistake Memory；History 补趋势/争议/重复作答；Bookmark 不进错因统计
Study Planner
  └─ Mistake + WeakPoint + LearningState → 每日计划
```

---

## 8. 通用字段约定

| 字段 | 类型 | 说明 |
|------|------|------|
| `created_at` | TIMESTAMPTZ | ISO 8601 |
| `updated_at` | TIMESTAMPTZ | 最后更新 |
| `user_id` | UUID | 多用户必填 |
| `metadata` | JSON | 扩展 |

主键统一 UUID。

---

## 9. API 分层建议（Web MVP）

| 层级 | 职责 |
|------|------|
| `POST /questions/analyze` | Question Coach |
| `POST /history` | QuestionHistory |
| `POST /bookmarks` | Bookmark |
| `POST /mistakes`（系统自动） | Mistake Coach |
| `GET /weak-points` | Mistake Coach |
| `POST /review/retrospective` | Review Coach |
| `GET/POST /study-plans` | Study Planner |
