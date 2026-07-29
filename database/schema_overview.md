# Database Schema Overview

> PMP AI Coach 共享数据层总览。定义多模块共用的实体、关系与读写边界。  
> **目标**：支撑 Skill MVP（`memory/` 映射）与未来 Web 产品（关系型 DB + API）无缝扩展。

---

## 1. 设计原则

| 原则 | 说明 |
|------|------|
| **单用户中心** | 所有学习数据以 `User` 为根，经 `user_id` 关联 |
| **模块共享写、分责读** | 无模块私有表；通过 Schema 文档约定各模块读写权限 |
| **枚举稳定** | `wrong_type`、`review_status` 等使用 snake_case 英文枚举，UI 层映射中文 |
| **单表精简** | MVP `Question` 仅 7 字段；`Mistake` 通过 `question_id` 关联 |
| **快照冗余** | `Mistake` 可保留精简快照；题目正文以 `Question` 为准 |
| **事实与推测分离** | 关键推断字段配 `confidence_level`；不确定不强行入库 |
| **Memory 映射** | `memory/*.md` 为 Agent 运行时视图，字段与下表一一对应 |

---

## 2. 实体关系（ER）

```
┌─────────────┐
│    User     │
└──────┬──────┘
       │ 1
       │
       ├──────────────────────────────────────────────┐
       │                                              │
       │ N                                            │ N
┌──────▼──────┐    N    ┌─────────────┐    N    ┌────▼────────┐
│  Mistake    │◄────────│  Question   │         │  WeakPoint  │
└──────┬──────┘    可选  └─────────────┘         └─────────────┘
       │                                              ▲
       │ N                                            │ 聚合自 Mistake
       │                                              │
┌──────▼──────────┐         ┌─────────────┐          │
│ ReviewSession   │         │  StudyPlan  │──────────┘
│  (含复习明细)    │         └──────┬──────┘
└─────────────────┘                │ 1
                                   │ N
                            ┌──────▼──────────┐
                            │ StudyPlanItem   │
                            └─────────────────┘

┌─────────────────┐
│LearningProgress │  ← 时间轴；汇总各模块活动
└─────────────────┘
```

---

## 3. Schema 文件索引

| 文件 | 核心实体 | 主要消费模块 |
|------|----------|--------------|
| `user_schema.md` | `User` | 全模块（读）；Study Planner（写偏好/阶段） |
| `question_schema.md` | `Question`（MVP 7 字段） | Question Coach（写）；Mistake（关联） |
| `mistake_schema.md` | `Mistake` | Question / Mistake / Review Coach |
| `learning_schema.md` | `WeakPoint`, `StudyPlan`, `StudyPlanItem`, `ReviewSession`, `LearningProgress` | Mistake / Review / Study Planner / Training Coach |

---

## 4. 模块读写矩阵

| 实体 | Question Coach | Mistake Coach | Review Coach | Study Planner | Training Coach |
|------|:--------------:|:-------------:|:------------:|:-------------:|:--------------:|
| `User` | R | R | R | R/W | R |
| `Question` | R/(W) | R | R | — | R |
| `Mistake` | W | R/W | R/W | R | — |
| `WeakPoint` | — | R/W | R | R | R |
| `StudyPlan` | — | — | R | R/W | — |
| `StudyPlanItem` | — | — | R/W | R/W | R |
| `ReviewSession` | — | — | R/W | R | — |
| `LearningProgress` | — | — | W | R/W | W |

R = 读，W = 写，R/W = 读写，(W) = 可选写（主题库归一化时）

---

## 5. Memory 层映射

Agent 在 Skill 环境下以 Markdown/JSON 维护个人记忆，与 Database 字段对齐：

| Memory 文件 | Database 实体 | 同步方向 |
|-------------|---------------|----------|
| `memory/user_profile.md` | `User` | 双向 |
| `memory/mistake_memory.md` | `Mistake[]` | DB → Memory（查询时）；保存时 Memory → DB |
| `memory/weak_points.md` | `WeakPoint[]` | Mistake 聚合后写入 |
| `memory/learning_progress.md` | `LearningProgress[]` | 各模块活动后追加 |

Web 产品实现时，`memory/` 可由 API 替代，Schema 不变。

---

## 6. 跨模块数据流

### 6.1 错题闭环

```
Question Coach
  └─ DATA_HANDOFF → Mistake（create/update）
Mistake Coach
  └─ aggregate → WeakPoint（upsert）
Study Planner
  └─ read WeakPoint + Mistake → StudyPlanItem（type=review）
Review Coach
  └─ execute → ReviewSession → update Mistake.review_status
  └─ append → LearningProgress
```

### 6.2 学习计划闭环

```
Study Planner
  └─ create StudyPlan + StudyPlanItem[]
  └─ write LearningProgress.next_plan
用户执行（Review / Training / Practice）
  └─ 各模块写 LearningProgress + 实体状态
Study Planner
  └─ read LearningProgress → 生成下一日计划
```

---

## 7. 通用字段约定

所有带 `user_id` 的实体建议包含：

| 字段 | 类型 | 说明 |
|------|------|------|
| `created_at` | TIMESTAMPTZ | 创建时间，ISO 8601 |
| `updated_at` | TIMESTAMPTZ | 最后更新时间 |
| `metadata` | JSON | 实验性扩展，未升维为列的字段 |

主键统一使用 UUID（`CHAR(36)` / `UUID` 类型）。

---

## 8. API 分层建议（Web MVP）

| 层级 | 职责 | 对应模块 |
|------|------|----------|
| `POST /questions/analyze` | 题目分析 | Question Coach |
| `POST /mistakes` `GET /mistakes` | 错题 CRUD | Mistake Coach |
| `GET /weak-points` | 薄弱点报告 | Mistake Coach |
| `POST /review-sessions` | 复习会话 | Review Coach |
| `GET/POST /study-plans` | 学习计划 | Study Planner |
| `GET/POST /learning-progress` | 进度时间轴 | 全模块 |
| `POST /materials/process` | 资料加工 | Training Coach |

路由命名与实体一致，便于前后端与 Agent 共用同一 Schema 文档。

---

## 9. 版本与迁移

| 字段 | 说明 |
|------|------|
| `schema_version` | 当前文档版本：`1.0.0` |
| 迁移策略 | 新增字段走 `metadata` 或 ALTER；枚举只增不改 |
| 考纲版本 | `User.exam_version` / `Question.exam_version` 支持多版本并存 |
