# PMP AI Coach Skill

**版本**：v0.1.1  
**类型**：可维护的 Agent Skill（文档 + 工作流 + 轻量 Memory），非传统应用仓库。

PMP AI Coach 帮助考生用 **单题深度分析 → 自动错题库 → 复盘 → 学习计划** 形成长期备考闭环。用户只与一个「PMP AI Coach」对话；内部分模块由 `skill.md` 静默路由。

---

## 项目介绍

本仓库定义：

| 层级 | 路径 | 作用 |
|------|------|------|
| Skill 入口 | [`skill.md`](skill.md) | 角色、路由优先级、全局约束 |
| 专业能力 | [`modules/`](modules/) | Question / Mistake / Review / Study Planner / Training Coach |
| 工作流 | [`workflows/`](workflows/) | 题目分析、错题入库、复盘、学习计划等执行步骤 |
| 数据契约 | [`database/`](database/) | Question、Mistake、History、Bookmark、Learning 等 Schema |
| 运行时记忆 | [`memory/data/`](memory/data/) | **本机** JSON（`.gitignore`）；映射 Schema，无数据库 |
| 知识库 | [`knowledge/`](knowledge/) | 考试框架、PMBOK、敏捷、决策树等 |
| 架构说明 | [`architecture/`](architecture/) | 数据流、Agent 流程 |
| 示例（公开） | [`examples/`](examples/) | 用户可见输出样例（Example Dataset，v0.1.0） |

设计原则：**可维护**（知识/流程/数据分离）、**可移植**（Markdown + JSON）、**可扩展**（预留 Web / DB）。

---

## 核心能力

### 1. 题目分析（Question Coach）

- 识别题干与选项，PMP 推理讲解，选项逐项分析  
- 有作答 → **自动写入** `question_history.json`  
- 答错 → **自动写入** `mistake_memory.json`（无需用户说「保存错题」）  
- 用户说「收藏这题」→ 仅 `bookmark_memory.json`  

工作流：[`workflows/question_capture.md`](workflows/question_capture.md) → [`workflows/question_analysis.md`](workflows/question_analysis.md)

### 2. 错题沉淀（Mistake Coach）

- P0：仅 `user_answer ≠ adjudication_answer` 入库（v0.1.1；题库标答为 `platform_answer`，争议不污染 Mistake）  
- 写入 `error_type`、`error_reason`、`knowledge_point`、`review_status`  
- 聚合 `weak_points.json`，追加 `learning_state.json`  

工作流：[`workflows/mistake_classification.md`](workflows/mistake_classification.md)

### 3. 学习复盘（Review Coach）

- 用户说「复盘」→ 从 **Mistake Memory** 重算错因（**不**读过期快照作输入）  
- **History**：答题趋势、重复作答、争议题（`answer_disputed`）  
- **Bookmark**：不参与错误统计  

工作流：[`workflows/review_retrospective.md`](workflows/review_retrospective.md)

### 4. 学习计划（Study Planner）

- 优先 **Mistake + WeakPoint + learning_state**  
- 可选融合 `review_retrospective.json`（仅当 `snapshot_status=current` 且指纹一致）  
- Bookmark 仅作考前辅助任务  

工作流：[`workflows/study_plan.md`](workflows/study_plan.md)

### 5. 概念学习（Training Coach）

- 无完整四选一题目时的概念讲解、资料加工  
- 不写 Mistake Memory  

---

## 使用方式

### 在 Cursor / Codex/ Workbuddy/ Agent 平台中启用

1. 将本仓库作为 Skill 或规则上下文加载。  
2. Agent **必须先读** [`skill.md`](skill.md)，再按意图加载 `workflows/` 与 `modules/`。  
3. 做题类交互写入 [`memory/data/`](memory/data/)（见下节）。

### 典型用户话术

| 用户说 | 行为 |
|--------|------|
| 粘贴题目 +「我选 A」 | Question Coach → History；若答错 → 自动 Mistake |
| 「复盘」 | Review Coach → 现场聚合 Mistake，可选更新复盘快照 |
| 「今天学什么」 | Study Planner |
| 「收藏这题」 | 仅 Bookmark（答对也可收藏） |
| 「什么是风险储备」 | Training Coach（非完整做题） |

### 输出示例

以下目录为**公开示例**（Example Dataset / 虚构 ID），**不是**仓库维护者的个人错题或学习记录：

见 [`examples/`](examples/)：

- [`wrong_question_example.md`](examples/wrong_question_example.md)  
- [`correct_question_example.md`](examples/correct_question_example.md)  
- [`review_report_example.md`](examples/review_report_example.md)  
- [`questions/mock_pmp_set_example_index.md`](examples/questions/mock_pmp_set_example_index.md)（题库目录加工示例）

模块级 JSON 样例：`modules/*/examples/`。  
Memory **结构**样例（可公开）：[`memory/data/examples/`](memory/data/examples/) — 仅供对照 Schema，勿与 `memory/data/*.json` 运行时混用。

---

## Memory 架构说明

**公开仓与个人隐私**：本仓库发布 **Skill 能力**（`skill.md`、`workflows/`、`knowledge/` 等）。**个人学习状态**仅存在于你本机的 `memory/data/*.json`（已 `.gitignore`），克隆仓库**不会**带入任何作者的错题、历史或画像。

| 路径 | 是否随 Git 发布 | 含义 |
|------|-----------------|------|
| [`examples/`](examples/) | **是** | 脱敏后的输出与流程演示（Example Dataset） |
| [`memory/data/examples/`](memory/data/examples/) | **是** | JSON **结构**样例，非真实用户数据 |
| `memory/data/*.json`（根目录） | **否** | 运行时：History、Mistake、Bookmark、WeakPoint 等，由你本地做题生成 |

MVP 采用 **三类记忆严格分离**（详见 [`memory/README.md`](memory/README.md)、[`database/schema_overview.md`](database/schema_overview.md)）：

```
用户做题（Question Coach）
        │
        ├──► question_history.json     每次有作答（对错均记）
        │
        ├──► mistake_memory.json       仅答错，自动
        │         └──► weak_points.json
        │
        └──► bookmark_memory.json     仅用户主动收藏

Review（复盘）── 重算 Mistake；History 补争议/趋势；不写 Bookmark 进错因
        └──► review_retrospective.json（写后快照，带 source_revisions）

Study Planner ── 优先 Mistake + WeakPoint
```

| 文件 | 实体 | 触发 |
|------|------|------|
| `memory/data/question_history.json` | QuestionHistory | 有作答自动 |
| `memory/data/mistake_memory.json` | Mistake | 答错自动 |
| `memory/data/bookmark_memory.json` | Bookmark | 主动收藏 |
| `memory/data/weak_points.json` | WeakPoint | Mistake 聚合 |
| `memory/data/learning_state.json` | 学习事件/统计 | 做题、复盘等 |
| `memory/data/review_retrospective.json` | 复盘快照 | 「复盘」后可选写入 |

上表文件均指 **`memory/data/` 根下运行时路径**（本地生成，默认不提交 Git）。

**字段规范（v0.1.0）**：新写入使用 `error_type`、`exam_domain`、`error_reason`；历史 JSON 中的 `mistake_type` / `eco_domain` 仅读兼容，不迁移。

### Memory 初始化

新用户从零启动、预置空 JSON、懒创建与个人数据隔离，见 **[`memory/INITIALIZATION.md`](memory/INITIALIZATION.md)**。

---

## 目录速查

```text
skill.md                 # Agent 主入口
modules/                 # 各 Coach 契约
workflows/               # 可执行工作流
database/                # Schema
memory/data/             # 运行时 JSON（本地 .gitignore，见 INITIALIZATION.md）
memory/data/examples/    # 公开 JSON 结构样例（Example Dataset）
memory/INITIALIZATION.md # Memory 从零初始化
examples/                # 公开 Markdown 输出示例（v0.1.0）
knowledge/               # 领域知识
inputs/                  # 输入识别
architecture/            # 架构与数据流
CHANGELOG.md             # 版本记录
```

---

## 版本与变更

当前发布：**v0.1.1** — 见 [`changelog.md`](changelog.md)。
