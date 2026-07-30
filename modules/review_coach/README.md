# Review Coach（复习教练模块）

> **所属产品**：PMP AI Coach（单一主 Skill）  
> **模块类型**：专业能力模块 | **非独立 Skill**

---

## 模块目标

负责 **错题与薄弱点的复习调度、复习执行引导与掌握度更新**，把「存下来的错题」转化为「真正少错」。

- 从 `mistake_memory` 筛选待复习题（`review_status` ≠ `mastered` / `ignored`）
- 结合 `weak_points` 生成「今日复习清单」
- 引导用户重做/自测并更新 `review_count`、`review_status`
- 输出单题或每日复习小结

**不负责**：首次题目讲解（→ Question Coach）、多周学习计划（→ Study Planner）、知识点精讲（→ Training Coach）。

---

## 用户触发场景

| 场景 | 典型用户说法 |
|------|--------------|
| 今日复习 | 「今天复习什么？」「错题本里哪些要复习？」 |
| 专项复习 | 「复习冲突管理相关错题」 |
| 复习打卡 | 「我刚复习了 3 道错题」 |
| 掌握确认 | 「这道题我掌握了」「可以标记 mastered」 |
| 间隔复习 | 「距上次复习一周了，提醒我」 |
| 考前冲刺复习 | 「考前帮我过一遍薄弱点」 |
| **学习复盘** | **「复盘」**（MVP：五段报告 + JSON） |

### MVP 实现（`mvp-1` — 复盘）

| 文件 | 说明 |
|------|------|
| `module.md` | 触发「复盘」、输入输出、Memory 读写 |
| `aggregation_rules.md` | 知识域/错因/薄弱点聚合规则 |
| `output_contract.md` | `REVIEW_RETROSPECTIVE_OUTPUT` |
| `examples/sample_retrospective_output.json` | 示例输出 |
| `workflows/review_retrospective.md` | 执行工作流 |

---

## 输入

| 输入类型 | 来源 | 说明 |
|----------|------|------|
| 错题记忆 | `memory/mistake_memory.md` | `review_status`、`repeated_count`、`last_reviewed_at` |
| 薄弱点 | `memory/weak_points.md` | 优先级与 `suggested_direction` |
| 用户档案 | `memory/user_profile.md` | 目标考试日期、每日学习时长 |
| 学习进度 | `memory/learning_progress.md` | 近期复习记录 |
| 用户反馈 | 对话 | 重做结果、掌握自评 |

---

## 输出

| 产出 | 格式 | 说明 |
|------|------|------|
| 今日复习清单 | Markdown 列表 | 题号/知识点/优先级/建议动作 |
| 单题复习引导 | 问答式 | 先自测 → 再揭示讲解要点 |
| 状态更新 | `mistake_memory` 字段变更 | `review_count++`、`review_status` 迁移 |
| 复习小结 | 简短报告 | 本次复习题数、仍薄弱标签 |
| 下一步建议 | 1–3 条 | 可交 Study Planner 衔接 |

---

## 数据依赖

### 工作流（Workflows）

| 文件 | 关系 |
|------|------|
| `workflows/review_retrospective.md` | **复盘**（用户说「复盘」） |
| `workflows/study_plan.md` | 复习任务生成（待完善，单题/日级） |
| `workflows/question_analysis.md` | 复习时错题再分析（可选） |

### 知识库（Knowledge）

| 路径 | 用途 |
|------|------|
| `knowledge/decision_framework/*` | 复习时强化判断路径 |
| `knowledge/language/pmp_terms.md` | 术语巩固 |
| `modules/question_coach/` 依赖的知识 | 错题重讲时引用 |

### 数据模型（Database）

| 文件 | 关系 |
|------|------|
| `database/mistake_schema.md` | `review_status` 状态机、`review_count` 规则 |

### 记忆（Memory）

| 文件 | 读写 |
|------|------|
| `memory/data/learning_state.json` | 读 |
| `memory/data/review_retrospective.json` | 读/写（最近一次复盘快照） |
| `memory/user_profile.md` | 读 |

### 协作模块

| 模块 | 关系 |
|------|------|
| `modules/mistake_coach/` | 错题数据来源 |
| `modules/question_coach/` | 复习时重新讲解 |
| `modules/study_planner/` | 日/周计划衔接 |

---

## 复习状态机（摘要）

```
new → learning → reviewing → mastered
  └────────────────────────→ ignored
```

详见 `database/mistake_schema.md` §4。

---

## 模块边界

```
mistake_memory + weak_points
        │
        ▼
Review Coach ──清单/引导/打卡──→ 用户
        │
        └──更新──→ mistake_memory.review_status
                    learning_progress
```
