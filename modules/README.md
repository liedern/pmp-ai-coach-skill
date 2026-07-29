# Modules（专业能力模块）

> **产品架构说明**：PMP AI Coach 保持 **一个主 Skill**（`skill.md`），内部分拆多个专业能力模块。  
> 各模块 **不是独立 Skill**，而是主 Skill 下的能力分区与路由索引。

---

## 设计原则

| 原则 | 说明 |
|------|------|
| **单一入口** | 用户只感知「PMP AI Coach」；模块由 Agent 按意图路由 |
| **共享基础设施** | `knowledge/`、`workflows/`、`database/`、`memory/`、`inputs/`、`architecture/` 全局共享 |
| **模块 = 能力 + 边界** | 每模块定义目标、触发、输入输出、数据依赖，不重复实现工作流正文 |
| **不删除既有内容** | 现有工作流与知识库文件保留，模块通过 README 引用 |

---

## 模块一览

| 模块 | 路径 | 核心职责 | 主工作流 |
|------|------|----------|----------|
| **Question Coach** | `modules/question_coach/` | 题目识别、PMP 推理、讲解、归档决策 | `workflows/question_analysis.md` |
| **Mistake Coach** | `modules/mistake_coach/` | 错题沉淀、错因归类、薄弱点聚合 | `workflows/mistake_classification.md` |
| **Review Coach** | `modules/review_coach/` | 错题复习调度、掌握度更新 | `workflows/study_plan.md`（复习部分） |
| **Study Planner** | `modules/study_planner/` | 学习计划、今日任务、进度闭环 | `workflows/study_plan.md` |
| **Training Coach** | `modules/training_coach/` | 概念讲解、资料加工、备考认知 | `skill.md` §6 + `material_processing.md` |

---

## 用户请求路由（Agent）

```
用户消息
    │
    ├─ 题目/截图/讲题/错题本 ──→ Question Coach
    │         └─ 保存 ──→ Mistake Coach
    │
    ├─ 薄弱点/错题统计 ────────→ Mistake Coach
    │
    ├─ 今日复习/复习打卡 ──────→ Review Coach
    │
    ├─ 学习计划/今天学什么 ────→ Study Planner
    │
    └─ 概念是什么/整理资料 ────→ Training Coach
```

详见 `architecture/agent_flow.md`。

---

## 第一版闭环（错题场景）

```
Question Coach → Mistake Coach → Review Coach / Study Planner
     ↑                                    │
     └──────── Training Coach（补知识）←─┘
```

---

## 目录结构

```text
PMP-Coach/
├── skill.md                 # 主 Skill（唯一入口）
├── modules/                 # 专业能力模块（本目录）
│   ├── README.md
│   ├── question_coach/
│   ├── mistake_coach/
│   ├── review_coach/
│   ├── study_planner/
│   └── training_coach/
├── workflows/               # 共享工作流（保留）
├── knowledge/               # 共享知识库（保留）
├── database/                # 共享数据模型（保留）
├── memory/                  # 共享个人记忆（保留）
├── inputs/                  # 共享输入层（保留）
├── architecture/            # 架构文档（保留）
└── source_materials/        # 原始资料（保留）
```

---

## 后续（待 Review 后）

- [ ] 在 `skill.md` 增加 `modules/` 路由索引（不改变主 Skill 身份）
- [ ] 各模块与 P0 模板/样例对齐
- [ ] `architecture/system_architecture.md` 增补 Module Layer 说明
