# Modules（专业能力模块）

> **产品架构说明**：PMP AI Coach 保持 **一个主 Skill**（`skill.md`），内部分拆多个专业能力模块。  
> 各模块 **不是独立 Skill**，而是主 Skill 下的能力分区与路由索引。  
> **用户永远只与「PMP AI Coach」对话**；模块由 `skill.md` §2 Module Router 自动分发，不向用户暴露模块名。

---

## 设计原则

| 原则 | 说明 |
|------|------|
| **单一入口** | 用户只感知「PMP AI Coach」；Agent 内部路由，禁止在回复中出现模块代号 |
| **共享数据层** | 各模块通过 `database/` 统一 Schema 读写用户学习数据，见 `database/schema_overview.md` |
| **共享基础设施** | `knowledge/`、`workflows/`、`memory/`、`inputs/`、`architecture/` 全局共享 |
| **模块 = 能力 + 边界** | 每模块定义目标、触发、输入输出、数据依赖；工作流正文不重复 |
| **不删除既有内容** | 现有工作流与知识库文件保留，模块通过 README 引用 |

---

## 模块一览

| 模块 | 路径 | 核心职责 | 主工作流 | 主数据实体 |
|------|------|----------|----------|------------|
| **Question Coach** | `modules/question_coach/` | 单题识别、PMP 推理、讲解、归档决策 | `workflows/question_analysis.md` | `Question`, `Mistake`（写入） |
| **Mistake Coach** | `modules/mistake_coach/` | 错题沉淀、错因归类、薄弱点聚合 | `workflows/mistake_classification.md` | `Mistake`, `WeakPoint` |
| **Review Coach** | `modules/review_coach/` | **执行**复习：选题、引导、打卡、掌握度 | `workflows/study_plan.md`（复习部分） | `ReviewSession`, `Mistake`（更新） |
| **Study Planner** | `modules/study_planner/` | **规划**学习：排期、今日任务、进度闭环 | `workflows/study_plan.md` | `StudyPlan`, `LearningProgress` |
| **Training Coach** | `modules/training_coach/` | 概念讲解、资料加工、备考认知 | `skill.md` §7 + `material_processing.md` | `LearningProgress`（可选） |

---

## 模块边界（关键）

### Question Coach vs Training Coach

两者都「讲 PMP」，但**触发条件、推理路径、输出结构**不同。Agent 必须先判定输入类型，再路由。

| 维度 | Question Coach | Training Coach |
|------|----------------|----------------|
| **触发信号** | 含完整题干 + 选项（文字/截图）；或用户要求「分析这道题」 | 概念提问、过程对比、ITTO、口诀；**无完整四选一题目** |
| **核心问题** | 「这道题选什么？我为什么错？」 | 「这个概念是什么？考试怎么考？」 |
| **推理框架** | PMP 五步 + `decision_tree` 六步决策链 | Teaching Mode 六段式（定义→考试意义→关键词→场景→陷阱→记忆法） |
| **必须执行** | `workflows/question_analysis.md` 全流程 | `skill.md` §7；不跑题目七步 |
| **错因分类** | 输出 `mistake_type`（8 类枚举） | 不涉及错因 |
| **数据写入** | 可写 `Mistake`（用户确认保存时） | 可选写 `LearningProgress`；不写 `Mistake` |
| **典型用户话** | 「我选了 A，答案是 B」「帮我看截图」 | 「Validate Scope 是什么？」「Risk 和 Issue 区别？」 |

**歧义消解规则**（Agent 内部）：

```
输入是否为题目截图，或含题干 + A/B/C/D 选项？
    ├─ 是 → Question Coach（即使措辞像「讲讲」）
    └─ 否 → 是否为概念/过程/对比类学习问题？
              ├─ 是 → Training Coach
              └─ 否 → 按 skill.md §2 继续判定其他意图
```

**协作**：Question Coach 讲解中发现用户概念薄弱 → 可**内嵌** Training Coach 式短讲解（≤1 段），但不切换输出模板；复习时 Review Coach 可调用 Training Coach 补知识。

---

### Review Coach vs Study Planner

两者都服务「复习与学习节奏」，但分工是 **执行 vs 规划**——类比「健身教练带你练」vs「教练给你排课表」。

| 维度 | Review Coach | Study Planner |
|------|--------------|---------------|
| **核心动词** | 复习、重做、打卡、掌握、间隔重复 | 计划、排期、今日任务、调整节奏 |
| **时间粒度** | **当下 / 单次会话**（现在复习哪几道题） | **日 / 周 / 备考周期**（这周学什么） |
| **输入侧重** | `Mistake.review_status`、`WeakPoint.priority_score` | `User.exam_target_date`、`LearningProgress`、薄弱域分布 |
| **输出侧重** | 复习清单 + 引导问答 + 更新 `review_count` / `review_status` | 学习计划表 + `next_plan` + 任务项 `StudyPlanItem` |
| **是否做题讲解** | 引导自测后可调用 Question Coach 重讲 | 不讲解，只分配任务类型与数量 |
| **典型用户话** | 「今天复习什么错题？」「这道题掌握了」 | 「帮我定两周计划」「今天学什么？」 |

**协作流程**：

```
Study Planner 生成 StudyPlanItem（`item_type=review_mistake`，含 mistake_ids）
        │
        ▼
Review Coach 消费任务项，执行复习会话 ReviewSession
        │
        ▼
回写 Mistake.review_status + LearningProgress
        │
        ▼
Study Planner 读取进度，生成下一日 next_plan
```

**边界红线**：

- Review Coach **不**制定多周备考策略
- Study Planner **不**引导单题重做过程
- 用户说「今天学什么」→ Study Planner 出任务包；其中 `item_type=review_mistake` / `weak_point_drill` 交给 Review Coach 执行

---

## Module Router（用户无感）

路由逻辑定义在 `skill.md` §2。P0 硬规则：

| 输入 | 路由 |
|------|------|
| 题目截图 / 题干 + 选项 | **Question Coach**（最高优先级） |
| 概念学习（无完整题目） | **Training Coach** |
| 同时含题目与概念 | **Question Coach**（概念仅内嵌补充） |

用户说「今天学什么」→ Study Planner；`item_type=review_mistake` 的任务执行时 → Review Coach。

---

## 数据流与共享 Schema

所有模块通过 `user_id` 关联同一用户的学习数据：

```
User
 ├── Question（共享题库，可选）
 ├── Mistake（Question Coach 写，Mistake/Review Coach 读写）
 ├── WeakPoint（Mistake Coach 聚合，Planner/Review 读）
 ├── StudyPlan + StudyPlanItem（Study Planner 写，Review 读）
 ├── ReviewSession（Review Coach 写）
 └── LearningProgress（Planner 写，全模块可读）
```

详见 `database/schema_overview.md`。

---

## 第一版闭环（错题场景）

```
Question Coach → Mistake（写入）
        │
        ▼
Mistake Coach → WeakPoint（聚合）
        │
        ├──────────────────┐
        ▼                  ▼
Study Planner          Review Coach
（排复习任务）          （执行复习）
        │                  │
        └────────┬─────────┘
                 ▼
         LearningProgress
                 │
                 ▼
    Training Coach（弱项补知识，按需）
```

---

## 目录结构

```text
PMP-Coach/
├── skill.md                 # 主 Skill（唯一入口 + Module Router）
├── modules/                 # 专业能力模块（本目录）
│   ├── README.md
│   ├── question_coach/
│   ├── mistake_coach/
│   ├── review_coach/
│   ├── study_planner/
│   └── training_coach/
├── workflows/               # 共享工作流
├── knowledge/               # 共享知识库
├── database/                # 共享数据模型（多模块读写）
│   ├── schema_overview.md   # 总览与 ER
│   ├── user_schema.md
│   ├── question_schema.md
│   ├── mistake_schema.md
│   └── learning_schema.md
├── memory/                  # Agent 运行时记忆（映射 database）
├── inputs/
├── architecture/
└── source_materials/
```

---

## MVP → Web 产品扩展

| 当前（Skill MVP） | 未来（Web 产品） |
|-------------------|------------------|
| `memory/*.md` 人工/Agent 维护 | 映射为 `database/` 表 + REST/GraphQL API |
| Module Router 在 `skill.md` 提示词 | 后端 `IntentService` + 各模块 Service |
| 单用户会话 | `User` 多租户 + 鉴权 |
| 工作流 Markdown | 工作流引擎 / 状态机（`review_status` 已有定义） |

Schema 字段命名与枚举自设计之初与 `mistake_schema.md` 对齐，便于无痛迁移。
