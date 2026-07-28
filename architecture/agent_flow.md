# Agent Flow

> PMP AI Coach Agent 工作流程定义：四大核心场景的触发、步骤与产出。
>
> **入口**：`skill.md` | **路由**：`inputs/*` | **详流**：`workflows/*`

---

## 0. Agent 总控流程

```
用户消息 / 附件
      │
      ▼
读取 skill.md（角色 + 推理框架 + 约束）
      │
      ▼
Input 识别（inputs/）
      │
      ▼
意图分类 ──┬── Question Answer
           ├── Wrong Question Analysis
           ├── Learning Plan
           └── Material Processing
      │
      ▼
加载对应 Workflow + 检索 Knowledge + 读取 Memory
      │
      ▼
执行并输出（templates/）+ 更新 Memory
```

---

## 1. Question Answer Flow（答疑流）

**触发**：用户询问 PMP 知识点、概念、过程区别，**非**完整做题场景。

| 步骤 | 动作 | 引用 |
|------|------|------|
| 1 | 识别为 `chat` + 教学意图 | `inputs/chat_input.md` |
| 2 | 进入 Teaching Mode | `skill.md` §6 |
| 3 | 检索相关 Knowledge | `knowledge/pmbok/`、`language/`、`decision_framework/` |
| 4 | 读取用户薄弱点（若有） | `memory/weak_points.md` |
| 5 | 结构化输出：定义 → 考试意义 → 关键词 → 场景 → 陷阱 → 记忆法 | `skill.md` |
| 6 | 可选更新 `learning_progress` | `memory/learning_progress.md` |

**不触发**：`question_analysis` 全量七步（除非用户贴完整题目）。

---

## 2. Wrong Question Analysis Flow（错题分析流）

**触发**：用户粘贴/上传题目、提供作答、要求讲解或加入错题本。

| 步骤 | 动作 | 引用 |
|------|------|------|
| 1 | 输入识别 | `inputs/question_input.md` 或 `image_input.md` |
| 2 | 执行 Question Analysis Workflow 全流程 | `workflows/question_analysis.md` |
| 3 | 决策推理 | `knowledge/decision_framework/decision_tree.md` |
| 4 | 优先级仲裁（多选项合理时） | `pmp_priority_rules.md` |
| 5 | 关键词扫描 | `knowledge/exam/exam_keywords_mapping.md`、`language/` |
| 6 | 输出固定结构 + DATA_HANDOFF | `workflows/question_analysis.md` §7–8 |
| 7 | 若保存：写 `mistake_memory`，聚合 `weak_points` | `memory/` |
| 8 | 个性化引用历史错题 | `memory/mistake_memory.md` |

```
题目输入 → question_analysis → decision_tree → 讲解输出
                    │
                    └→ mistake_memory → weak_points
```

---

## 3. Learning Plan Flow（学习计划流）

**触发**：用户要求制定/调整学习计划、今日学什么、考前冲刺安排。

| 步骤 | 动作 | 引用 |
|------|------|------|
| 1 | 识别为 `chat` + 规划意图 | `inputs/chat_input.md` |
| 2 | 读取用户档案 | `memory/user_profile.md`（目标日期、阶段、偏好） |
| 3 | 读取薄弱点 | `memory/weak_points.md` |
| 4 | 读取近期进度 | `memory/learning_progress.md` |
| 5 | 读取待复习错题 | `memory/mistake_memory.md`（review_status ≠ mastered） |
| 6 | 执行 Study Plan Workflow（待完善） | `workflows/study_plan.md` |
| 7 | 输出可执行计划（日/周） | `templates/study_plan.md`（待完善） |
| 8 | 写入 `learning_progress.next_plan` | `memory/learning_progress.md` |

**优先级逻辑**：

```
距考试天数少 → 模考 + 错题 + 弱项
weak_points.priority_score 高 → 优先排期
user_profile.learning_style → 匹配内容形态（视频/刷题/阅读）
```

---

## 4. Material Processing Flow（资料加工流）

**触发**：用户上传 PDF/Word/Excel/讲义/术语表/题库文件。

| 步骤 | 动作 | 引用 |
|------|------|------|
| 1 | 输入识别 | `inputs/document_input.md` |
| 2 | 执行 Material Processing 全流程 | `workflows/material_processing.md` |
| 3 | 分类 → 提取 → 结构化 → 质检 | Workflow §3–7 |
| 4 | 双语处理 | `knowledge/language/` 规范 |
| 5 | 写入 Knowledge 目标路径 | `architecture/data_flow.md` §2.1 |
| 6 | 输出加工报告 + MATERIAL_HANDOFF | Workflow §4.7 |
| 7 | Need Review 项不静默入库 | 人工确认队列 |

```
PDF/Word → material_processing → knowledge/
                      │
                      └→ imported_materials 索引
```

**子路由**：

| 资料类型 | 目标 |
|----------|------|
| 题库 | `examples/questions/` + 可选触发 question_analysis |
| 术语表 | `knowledge/language/` |
| 讲义口诀 | `knowledge/decision_framework/` |

---

## 5. 流程选择速查

| 用户信号 | 流程 |
|----------|------|
| 「这道题」「选错了」「帮我讲题」 | Wrong Question Analysis |
| 「什么是」「解释一下」「Risk 和 Issue 区别」 | Question Answer |
| 「学习计划」「今天学什么」「考前安排」 | Learning Plan |
| 「上传」「导入」「整理这份资料」 | Material Processing |
| 「模考」「模拟考」 | Mock Exam（`workflows/mock_exam.md`，待完善） |
| 「我哪弱」「错误模式」 | mistake_classification + weak_points 聚合 |

---

## 6. Agent 跨流程协作

| 协作点 | 说明 |
|--------|------|
| 错题积累 → 学习计划 | `weak_points` 驱动 `study_plan` 优先级 |
| 资料入库 → 答疑 | 新术语进入 `language/` 后可被 Question Answer 引用 |
| 学习进度 → 用户档案 | 阶段达标时更新 `user_profile.current_study_stage` |
| 加工题库 → 错题分析 | `examples/questions/` 条目可进入 question_analysis |
