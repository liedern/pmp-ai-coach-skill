# Study Planner（学习规划模块）

> **所属产品**：PMP AI Coach（单一主 Skill）  
> **模块类型**：专业能力模块 | **非独立 Skill**  
> **版本**：`mvp-1`

---

## 模块目标

将 **错题记录、`weak_points`、学习状态**（及 Review 复盘快照）转化为 **每日学习计划**。

**不负责**：单题讲解（→ Question Coach）、复习执行（→ Review Coach）。

---

## 用户触发场景

| 场景 | 典型用户说法 |
|------|--------------|
| 今日任务 | 「今天学什么？」「今日学习计划」 |
| 制定计划 | 「帮我安排今天」「考前怎么学」 |
| 带约束 | 「只有 1 小时」「重点变更管理」 |

---

## 输入（MVP）

| 文件 | 说明 |
|------|------|
| `memory/data/mistake_memory.json` | 错题与收藏 |
| `memory/data/weak_points.json` | Review Coach / Mistake Coach 薄弱点 |
| `memory/data/learning_state.json` | 阶段、错误模式、会话 |
| `memory/user_profile.json` | 考试日期、每日时长（可选） |
| `memory/data/review_retrospective.json` | 最近一次复盘（可选） |

---

## 输出

| 产出 | 说明 |
|------|------|
| 每日计划 Markdown | 六要素 + 任务清单 |
| `STUDY_PLAN_HANDOFF` JSON | 见 `output_contract.md` |
| 可选 | `memory/data/daily_study_plan.json` |

---

## MVP 实现文件

| 文件 | 说明 |
|------|------|
| `module.md` | 定位、输入输出、流程 |
| `output_contract.md` | `daily_plan` 六要素 + `study_plan_items` |
| `planning_rules.md` | 优先级、题量、ECO 权重、考试倒计时 |
| `examples/sample_study_plan.json` | 完整示例 |
| `workflows/study_plan.md` | 执行工作流 |

---

## 模块边界

```
mistake_memory + weak_points + learning_state
        │
        ▼
Study Planner ──每日计划──→ 用户
        │
        ├── review_mistake ──→ Review Coach
        └── knowledge_learning ──→ Training Coach
```
