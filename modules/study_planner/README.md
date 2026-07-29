# Study Planner（学习规划模块）

> **所属产品**：PMP AI Coach（单一主 Skill）  
> **模块类型**：专业能力模块 | **非独立 Skill**

---

## 模块目标

负责 **将备考目标、薄弱点与进度转化为可执行的学习计划**，回答「接下来学什么、学多少、按什么顺序」。

- 读取考试目标日期与学习阶段，倒推备考节奏
- 综合 `weak_points`、`mistake_memory` 排定优先级
- 生成日/周级学习任务（复习、刷题、阅读、模考）
- 写入 `learning_progress.next_plan` 形成闭环

**不负责**：单题深度讲解（→ Question Coach）、复习过程引导（→ Review Coach）、资料 OCR 加工（→ Training Coach / material_processing）。

---

## 用户触发场景

| 场景 | 典型用户说法 |
|------|--------------|
| 制定计划 | 「帮我制定两周学习计划」「考前 30 天怎么安排？」 |
| 今日任务 | 「今天学什么？」「今日任务是什么？」 |
| 调整计划 | 「我时间不够，压缩一下计划」 |
| 进度汇报后更新 | 「今天做了 30 道题，正确率 75%」 |
| 弱项突破 | 「我风险管理最弱，这周重点攻这个」 |
| 考前冲刺 | 「还有 14 天考试，怎么冲刺？」 |

---

## 输入

| 输入类型 | 来源 | 说明 |
|----------|------|------|
| 用户档案 | `memory/user_profile.md` | `exam_target_date`、`current_study_stage`、`learning_style` |
| 薄弱点 | `memory/weak_points.md` | `priority_score`、`suggested_direction` |
| 错题记忆 | `memory/mistake_memory.md` | 待复习题数量与领域分布 |
| 学习进度 | `memory/learning_progress.md` | 近期完成率、`still_weak` |
| 用户指令 | 对话 | 可用时间、偏好、约束 |

---

## 输出

| 产出 | 格式 | 说明 |
|------|------|------|
| 学习计划 | Markdown（日/周） | 可执行任务列表，含时长与数量 |
| 今日任务 | 3–5 条 | 复习 + 练习 + 阅读组合 |
| 优先级说明 | 简短理由 | 引用 weak_points / 考试倒计时 |
| 进度记录 | `learning_progress` 新条目 | `study_date`、`next_plan` |
| 可选提醒 | `user_profile.reminders` | 复习/模考提醒项 |

---

## 数据依赖

### 工作流（Workflows）

| 文件 | 关系 |
|------|------|
| `workflows/study_plan.md` | **主执行流程**（待完善） |
| `workflows/mock_exam.md` | 模考排期（未来，P1） |

### 知识库（Knowledge）

| 路径 | 用途 |
|------|------|
| `knowledge/exam/ECO.md` | 领域权重（待填充） |
| `knowledge/pmbok/*` | 推荐阅读章节 |
| `knowledge/decision_framework/*` | 口诀与规则专项 |

### 数据模型（Database）

| 文件 | 关系 |
|------|------|
| `database/learning_schema.md` | 学习进度持久化（待完善） |
| `database/mistake_schema.md` | 错题复习优先级参考 |

### 记忆（Memory）

| 文件 | 读写 |
|------|------|
| `memory/user_profile.md` | **读**（目标、阶段、偏好） |
| `memory/weak_points.md` | **读** |
| `memory/mistake_memory.md` | 读 |
| `memory/learning_progress.md` | **读/写** |

### 协作模块

| 模块 | 关系 |
|------|------|
| `modules/mistake_coach/` | 薄弱域输入 |
| `modules/review_coach/` | 输出复习任务 → Review 执行 |
| `modules/training_coach/` | 输出阅读/概念任务 |

---

## 第一版范围建议

P0 仅实现 **单题级复习建议**（Question Coach 输出后 3 条动作）与 **今日任务 3–5 条**，不做完整多周算法排期。

---

## 模块边界

```
user_profile + weak_points + mistake_memory
        │
        ▼
Study Planner ──计划/今日任务──→ 用户
        │
        ├──触发──→ Review Coach（复习项）
        ├──触发──→ Training Coach（阅读项）
        └──写入──→ learning_progress
```
