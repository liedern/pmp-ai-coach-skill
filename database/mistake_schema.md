# Mistake Schema

> 用户个人错题记录（Mistake Record）数据模型设计文档。
>
> 本文档定义产品级字段、枚举与状态机，**不是**可执行代码或 ORM 定义。具体存储引擎（PostgreSQL、SQLite、文档库等）由实现层选择。

---

## 1. Purpose

`Mistake`（Mistake Memory）是 PMP AI Coach 中**以用户为中心**的核心实体，**仅**记录「某用户在某次做题中**答错**」及其分析结果与复习进度。

> **三分流硬规则**（与 Bookmark / History 严格分离）：
>
> | 存储 | 触发 | 禁止 |
> |------|------|------|
> | **Mistake Memory** | `user_answer` ≠ **`adjudication_answer`**（v0.1.1；legacy = `correct_answer`）→ **自动**入库 | 答对、仅收藏、仅讲解、争议且用户=Coach |
> | Bookmark Memory | 用户主动收藏 | 不得因答错自动写入收藏 |
> | Question History | 凡做过的题均记一笔 | 不得替代 Mistake |

该数据用于：

| 用途 | 说明 |
|------|------|
| **错题复盘** | Review Coach **只读** Mistake Memory |
| **错因 / 高频错误模式** | 按 `error_type`、`knowledge_point`、`exam_domain` 聚合 |
| **薄弱知识点识别** | Mistake Coach → WeakPoint |
| **学习计划生成** | Study Planner **优先**读 Mistake Memory |
| **Web / 多用户** | `user_id` 必填；字段命名稳定，便于 DB 接入 |

### Mistake Record（产品最小字段）

| 字段 | 必须 | 说明 |
|------|:----:|------|
| `mistake_id` | 是 | 主键 |
| `question_id` | 是 | 关联 Question |
| `user_id` | 是 | 多用户隔离 |
| `question_text` | 是 | 题干快照 |
| `options` | 是 | 选项快照 |
| `user_answer` | 是 | 用户答案 |
| `correct_answer` | 是 | 当次 **判题基准**（= `adjudication_answer`；通常为 Coach 判断） |
| `error_type` | 是 | 见 §3.4 产品枚举 |
| `error_reason` | 是 | 错因说明 |
| `knowledge_point` | 是 | 知识点标签（数组或主标签） |
| `exam_domain` | 推荐 | ECO：`people` / `process` / `business_environment` |
| `review_status` | 是 | 默认 `new` |
| `created_at` | 是 | ISO 8601 |

**历史只读别名**：`mistake_type`、`mistake_reason`、`eco_domain` — **禁止新写入**；读取时回退至 canonical（见 `schema_overview.md` §1.1）。

### 与其他实体的关系

```
User (1) ──< (N) Mistake
Question (1) ──< (N) Mistake
User (1) ──< (N) QuestionHistory   # 见 question_schema.md
User (1) ──< (N) Bookmark          # 见 question_schema.md；≠ Mistake
```

- **Mistake** = 答错 + 错因 + 复习状态  
- **Bookmark** ≠ Mistake（答对也可收藏）  
- **QuestionHistory** = 每次作答轨迹（对/错均记）

### 设计原则

1. **事实与推测可区分**：`correct_answer`、`wrong_type` 等可附 `confidence_level`；不确定时不强行填错因
2. **快照冗余**：`question_text`、`options` 在 Mistake 上保留快照，避免主题库变更后用户记录失真
3. **枚举稳定**：`error_type`（MVP）、`wrong_type`（扩展）、`review_status` 使用固定英文枚举，便于跨语言 UI 与 API
4. **Review Coach 可读**：Mistake Coach 入库时填充 §2.9 字段，Review Coach 按 `error_type` + `knowledge_point` 聚合，无需额外推断
5. **可扩展**：§5 预留扩展字段，核心表保持精简

---

## 2. Core Fields

下表列出 Mistake 记录的核心字段。类型建议以关系型数据库为基准，NoSQL 实现时可等价映射。

### 2.1 标识与关联

| 字段名称 | 类型建议 | 是否必须 | 用途 |
|----------|----------|----------|------|
| `mistake_id` | UUID / BIGINT | **是** | 错题记录主键，全局唯一 |
| `user_id` | UUID / BIGINT | **是** | 所属用户 ID，外键关联 User |
| `question_id` | UUID / BIGINT | **是**（MVP） | 外键关联 `Question.question_id`；保存错题前须先创建/匹配 Question |

### 2.2 题目来源与快照

| 字段名称 | 类型建议 | 是否必须 | 用途 |
|----------|----------|----------|------|
| `source` | VARCHAR(255) | 否 | 题目来源（如 PMBOK、某培训机构题库、真题、自编）。未知为 `null` |
| `exam_set` | VARCHAR(255) | 否 | 套卷名称或编号（如「第 3 套模拟题 Q12」），便于按套卷复盘 |
| `question_text` | TEXT | **是** | 题干全文快照（含场景描述），保存时固化，不随主题库变更 |
| `options` | JSON / JSONB | **是** | 选项快照，键值对格式：`{"A":"...","B":"...","C":"...","D":"..."}`。兼容多选时值为数组或增加 `option_type` 扩展 |

### 2.3 作答与讲解

| 字段名称 | 类型建议 | 是否必须 | 用途 |
|----------|----------|----------|------|
| `user_answer` | VARCHAR(32) | 否 | 用户本次作答。单选如 `"B"`；多选如 `["A","C"]`（JSON 数组）。未提供为 `null`。**Review Coach** 用于判断对错与错因是否有效 |
| `correct_answer` | VARCHAR(32) | 否 | 正确答案。用户或官方提供为【事实】；仅 Agent 推断时仍入库但应配合 `confidence_level`。不确定为 `null`。**Review Coach** 与 `user_answer` 对比计算确认错题 |
| `answer_explanation` | TEXT / JSON | 否 | 答案讲解。可为纯文本，或结构化 JSON（对齐 `question_analysis` 的 `explanation` 对象：含 `correct_answer_reason`、`option_analysis`、`memory_rule` 等） |

### 2.4 PMP 分析维度

| 字段名称 | 类型建议 | 是否必须 | 用途 |
|----------|----------|----------|------|
| `project_type` | ENUM / VARCHAR(32) | 否 | 项目类型：`predictive` / `agile` / `hybrid` / `unknown` |
| `project_phase` | ENUM / VARCHAR(32) | 否 | 项目阶段：`initiating` / `planning` / `executing` / `monitoring_controlling` / `closing` / `unknown` |
| `exam_domain` | ENUM / VARCHAR(32) | 否 | ECO 领域（**读兼容** `eco_domain`） |
| `knowledge_points` | JSON / TEXT[] | 否 | 具体知识点标签列表（复数，关系型/API 常用），如 `["冲突管理","合作/解决问题"]` |
| `knowledge_point` | JSON / TEXT[] | 否 | **与 `knowledge_points` 同义**，Mistake Coach / `memory/mistake_memory.md` / **Review Coach** 聚合用；实现层二选一写入，读取时合并 |
| `process` | VARCHAR(128) | 否 | PMBOK 过程名称或敏捷实践名（如 `实施整体变更控制`、`Sprint Planning`），比 `knowledge_point` 更贴近考纲过程粒度 |

### 2.5 错因与难度

| 字段名称 | 类型建议 | 是否必须 | 用途 |
|----------|----------|----------|------|
| `error_type` | ENUM / VARCHAR(64) | 否 | **MVP 错因主分类**（Question Coach / Mistake Coach / **Review Coach** 首选字段），枚举见 **§3.4** |
| `error_reason` | TEXT | 否 | 错因自然语言说明（MVP 首选）；与 `wrong_reason` **同义**，内容一致 |
| `wrong_type` | ENUM / VARCHAR(64) | 否 | 扩展错因分类，枚举见 §3.1；持久化层可与 `error_type` 映射（§3.5） |
| `wrong_reason` | TEXT | 否 | 与 `error_reason` 同义；历史字段名，新写入建议同步填充 `error_reason` |
| `difficulty` | ENUM / SMALLINT | 否 | 题目难度：`easy` / `medium` / `hard`，或 1–5 数值。可由用户标记、题库元数据或 AI 估算 |
| `confidence_level` | ENUM / VARCHAR(16) | 否 | 用户对本次作答的自信程度，或系统对 `correct_answer` / `error_type` 推断置信度：`high` / `medium` / `low` |

### 2.6 复习状态与统计

| 字段名称 | 类型建议 | 是否必须 | 用途 |
|----------|----------|----------|------|
| `review_status` | ENUM / VARCHAR(32) | **是** | 复习生命周期状态，枚举见 §4。新入库默认 `new`。**Review Coach** 用于待复习清单与复盘中的掌握度视图 |
| `review_count` | INTEGER | **是** | 累计复习次数（含主动复习与系统推送复习），默认 `0` |
| `created_at` | TIMESTAMPTZ | **是** | 记录创建时间（首次入库），ISO 8601 |
| `updated_at` | TIMESTAMPTZ | **是** | 记录最后更新时间（含状态变更、追加复习、更新讲解） |

### 2.7 字段约束摘要

| 规则 | 说明 |
|------|------|
| **真实错题门槛（P0）** | 仅当 `user_answer` 与 **`adjudication_answer`** 均非 `null`，且规范化后 **不相等**，且通过 `decision_rules.md` §0.2 争议门禁时，才允许创建/更新 Mistake |
| 入库最低要求 | `mistake_id`、`user_id`、`question_text`、`options`、`user_answer`、`correct_answer`、`error_type`、`error_reason`、`knowledge_point`、`review_status`、`review_count`、`created_at`、`updated_at` |
| 无用户答案 | `user_answer = null` 时，**不得入库**；`error_type` 与 `wrong_type` 必须为 `null` |
| 答对 | `user_answer === correct_answer` 时，**不得入库**（含用户要求「加入错题本」） |
| 错因字段一致 | 若同时写入 `error_type` 与 `wrong_type`，须满足 §3.5 映射；`error_reason` 与 `wrong_reason` 内容一致 |
| 知识点字段一致 | `knowledge_point` 与 `knowledge_points` 内容一致（或仅写其一，读取端合并） |
| Review Coach 聚合 | 仅以满足 **P0 真实错题** 的记录统计；`error_type` 非 `null` |
| 重复题目 | 同一 `user_id` + 同一 `question_id`（或题干指纹相同）→ 更新已有 Mistake，递增 `review_count`，刷新 `updated_at`，不重复创建 |
| 选项格式 | `options` 至少包含一个键值对；键名推荐 `A`–`D` 或 `1`–`4`，全库保持一致 |

### 2.8 与 Question Analysis 的字段映射

| Data Handoff 字段 | Mistake 字段 |
|-------------------|--------------|
| `question_text` | `question_text` |
| `options` | `options` |
| `user_answer` | `user_answer` |
| `correct_answer` | `correct_answer` |
| `source` | `source` |
| `exam_set` | `exam_set` |
| `project_approach` | `project_type` |
| `project_phase` | `project_phase` |
| `exam_domain` / `eco_domain`（别名） | `exam_domain` |
| `knowledge_points` | `knowledge_points` / `knowledge_point`（同值） |
| `error_type` / `mistake_type`（别名） | `error_type`（MVP）；并映射 `wrong_type`（§3.5） |
| `error_reason` / `mistake_reason`（别名） | `error_reason` / `wrong_reason`（同值） |
| `explanation` | `answer_explanation` |
| `correct_answer_confidence` | `confidence_level`（推断置信度时） |
| 工作流 `review_status` | 入库后映射为 Mistake `review_status`（见 §4.3） |

### 2.9 Review Coach 分析字段（MVP 摘要）

Mistake Coach 写入、Review Coach 读取时，**真实错题**须包含以下字段（`memory/data/mistake_memory.json` 与 Schema 对齐）：

| 字段 | 类型 | 真实错题必填 | Review Coach 用途 |
|------|------|:------------:|-------------------|
| `user_answer` | string / array | **是** | 作答快照 |
| `correct_answer` | string / array | **是** | 对错判定 |
| `error_type` | enum §3.4 | **是** | 高频错误类型分析 |
| `error_reason` | text | **是** | 错因说明、模式摘要 |
| `knowledge_point` | string[] | **是** | 知识领域排序（与 `knowledge_points` 同值） |
| `review_status` | enum §4 | **是** | 待复习清单、掌握度 |

记忆层：**新入库仅写** `error_type`、`error_reason`、`exam_domain`。Review / Planner 读取时 `error_type ?? mistake_type`。

---

## 3. Error Type & Wrong Type

MVP 链路（Question Coach → Mistake Coach → **Review Coach**）以 **`error_type`** 为错因主字段；关系型持久化或历史集成可同时写入 **`wrong_type`**（扩展枚举，§3.1）。

### 3.1 `wrong_type` 扩展枚举一览

| 存储值 | 英文名称 | 中文说明 |
|--------|----------|----------|
| `knowledge_gap` | Knowledge Gap | 知识盲区 |
| `concept_confusion` | Concept Confusion | 概念混淆 |
| `scenario_judgment_error` | Scenario Judgment Error | 场景判断错误 |
| `process_sequence_error` | Process Sequence Error | 流程顺序错误 |
| `role_and_responsibility_error` | Role and Responsibility Error | 角色与职责错误 |
| `terminology_problem` | Terminology Problem | 术语理解问题 |
| `carelessness` | Carelessness | 粗心 |
| `insufficient_information` | Insufficient Information | 信息不足 |

### 3.2 扩展分类详解（`wrong_type`）

#### Knowledge Gap（知识盲区）

- **定义**：用户不了解该题所考的知识点、过程、工具或敏捷实践，无法建立正确判断基础。
- **典型表现**：未掌握变更控制流程、不熟悉冲突解决策略、不知道某角色的职责边界。
- **与学习建议关联**：优先补充教材/过程组讲解，再做同知识点专项练习。

#### Concept Confusion（概念混淆）

- **定义**：掌握部分知识，但将两个相近概念、过程或工件混为一谈。
- **典型表现**：风险 vs 问题、章程 vs 项目管理计划、缺陷 vs 变更请求、赶工 vs 快速跟进。
- **与学习建议关联**：对比表记忆、易混概念专项刷题。

#### Scenario Judgment Error（场景判断错误）

- **定义**：理解概念，但误判题干情境、约束条件、相关方诉求或冲突严重程度。
- **典型表现**：在应协商时选择强制；在应升级时过度自行处理；忽略「全球团队」「监管」等场景约束。
- **与学习建议关联**：强化场景关键词识别与「情境 → 行动」映射训练。

#### Process Sequence Error（流程顺序错误）

- **定义**：知道相关概念，但答题时步骤或先后顺序错误，违反 PMP 决策逻辑。
- **典型表现**：先执行后批准、先关闭后验收、未更新计划即执行、跳过分析直接行动。
- **与学习建议关联**：记忆标准处理顺序（Analysis → Collaborate → Process → Plan）。

#### Role and Responsibility Error（角色与职责错误）

- **定义**：混淆项目经理、产品负责人、Scrum Master、发起人、团队成员等角色的职责与权限。
- **典型表现**：PM 替 PO 排优先级；SM 直接分派任务；未经发起人批准即变更章程范围。
- **与学习建议关联**：角色职责矩阵复习，区分 Predictive 与 Agile 语境下的同一头衔。

#### Terminology Problem（术语理解问题）

- **定义**：因英文题干关键词或专业术语理解偏差导致选错，而非完全不会该知识点。
- **典型表现**：误解 `mitigate` / `transfer`、`validate` / `verify`、否定词 `NOT` / `EXCEPT`。
- **与学习建议关联**：术语表背诵、题干关键词划词练习。

#### Carelessness（粗心）

- **定义**：具备判断能力，但因审题不细、手滑选错、忽略限定词等非知识性原因答错。
- **典型表现**：选成相邻选项、漏看「第一步」「最不恰当」、多选当单选。
- **与学习建议关联**：考场审题 checklist，同题二次作答验证。

#### Insufficient Information（信息不足）

- **定义**：题干、选项或用户提供的材料不足以可靠判断正确答案或错因；Agent 也不应强行推断。
- **典型表现**：OCR 残缺、选项缺失、截图模糊、用户未提供自己的答案却要求判错因。
- **与学习建议关联**：提示用户补充信息；`error_type` / `wrong_type` 可为 `null` 直至信息完整。

### 3.3 分类使用规则

| 条件 | `error_type` | `wrong_type` |
|------|--------------|--------------|
| 用户未提供 `user_answer` | `null` | `null` |
| 用户答对且无疑义 | `null` | `null`（**不写 Mistake**；写 History；收藏另走 Bookmark） |
| 用户答错或自述做错 | 必选 §3.4 一项，`error_reason` 必填 | 按 §3.5 映射写入 |
| 信息不足无法分类 | `null` 或暂不填 | 可为 `insufficient_information` |

### 3.4 `error_type` 枚举（产品 — Review Coach 聚合）

**Canonical 字段名：`error_type`**（**历史只读**：`mistake_type`）。主类型取以下之一：

| 存储值 | 英文名称 | 中文说明 |
|--------|----------|----------|
| `knowledge_gap` | Knowledge Gap | 知识缺失 |
| `concept_confusion` | Concept Confusion | 概念混淆 |
| `careless_error` | Careless Error | 粗心 |
| `question_reading_error` | Question Reading Error | 题干理解错误 |
| `trap_option_error` | Trap Option Error | 选项陷阱 |

存储值使用 **snake_case**。Review Coach 的 `error_type_analysis.by_type` **按 `error_type` 分组**（兼容读取 `mistake_type`）。

**兼容映射**（旧六项 → 产品五项）：

| 旧值 | 新产品值 |
|------|----------|
| `knowledge_gap` | `knowledge_gap` |
| `concept_confusion` | `concept_confusion` |
| `scenario_judgment_error` | `trap_option_error` |
| `process_order_error` | `concept_confusion` |
| `keyword_misread` | `question_reading_error` |
| `careless` | `careless_error` |

### 3.5 `error_type` ↔ `wrong_type` 映射

关系型层可同时写 `error_type`（产品）与 `wrong_type`（扩展库表）；**禁止**新写 `mistake_type` 别名。

| `error_type`（产品） | `wrong_type`（扩展） |
|------------------------|----------------------|
| `knowledge_gap` | `knowledge_gap` |
| `concept_confusion` | `concept_confusion` |
| `careless_error` | `carelessness` |
| `question_reading_error` | `terminology_problem` |
| `trap_option_error` | `scenario_judgment_error` |

**读取规则**：Review Coach 以 `error_type` 为准（回退 `mistake_type`）；若仅有 `wrong_type`，按上表归一后再聚合。`error_reason` 与 `mistake_reason` / `wrong_reason` 视为同一字段。

## 4. Review Status

`review_status` 描述错题在**个人复习生命周期**中的位置，与「是否做错」正交：一条记录一旦入库，通过状态机跟踪从「新错题」到「已掌握」或「已忽略」。

### 4.1 状态枚举

| 存储值 | 英文 | 中文 | 含义 |
|--------|------|------|------|
| `new` | New | 新错题 | 刚入库，尚未开始系统复习 |
| `learning` | Learning | 学习中 | 用户已开始针对本题或相关知识点的学习 |
| `reviewing` | Reviewing | 复习中 | 已至少复习一次，仍未达到掌握标准 |
| `mastered` | Mastered | 已掌握 | 连续复习达标或重做正确且用户确认掌握 |
| `ignored` | Ignored | 已忽略 | 用户主动跳过或标记为不再复习（如判定为无效题） |

### 4.2 状态变化逻辑

```
                    ┌─────────────┐
                    │    new      │  ← 默认入库状态
                    └──────┬──────┘
                           │
           ┌───────────────┼───────────────┐
           │               │               │
           ▼               ▼               ▼
    ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
    │  learning   │ │  reviewing  │ │   ignored   │
    └──────┬──────┘ └──────┬──────┘ └─────────────┘
           │               │
           └───────┬───────┘
                   │
                   ▼
            ┌─────────────┐
            │  mastered   │
            └─────────────┘
```

#### 转换规则

| 从 | 到 | 触发条件 |
|----|-----|----------|
| `new` | `learning` | 用户查看讲解并开始学习（如点击「开始学习」）；或 `review_count` 首次增加且用户完成学习动作 |
| `new` | `reviewing` | 用户完成第一次复习打卡（`review_count >= 1`） |
| `new` | `ignored` | 用户标记「忽略」「不是有效错题」 |
| `learning` | `reviewing` | 完成至少一次复习行为，`review_count` 递增 |
| `learning` | `mastered` | 重做正确 + 间隔复习达标（见下方掌握规则） |
| `learning` | `ignored` | 用户主动忽略 |
| `reviewing` | `mastered` | 满足掌握规则 |
| `reviewing` | `learning` | 重做再次做错，掌握度回退，需重新学习 |
| `reviewing` | `ignored` | 用户主动忽略 |
| `mastered` | `reviewing` | 间隔很久后重做错误，或用户手动「取消掌握」 |
| `ignored` | `new` / `learning` | 用户恢复复习 |
| `mastered` | `ignored` | 用户忽略（少见，保留路径） |

#### 掌握（`mastered`）判定建议

满足以下**任一**条件可转为 `mastered`（产品可配置）：

1. 同一题在复习场景下**连续 2 次**作答正确，且间隔 ≥ 3 天
2. 用户主动标记「已掌握」且最近一次重做正确
3. `review_count >= 3` 且最近一次重做正确，且 `error_type` 非 `careless` / `wrong_type` 非 `carelessness` 时需额外确认

#### `review_count` 递增时机

- 用户完成一次「复习打卡」（查看讲解 + 自测 / 重做）
- 系统推送的间隔复习完成
- **不**因仅打开列表或只读查看而递增（可另设 `view_count` 扩展）

### 4.3 与工作流入库状态的映射

`question_analysis` 工作流使用入库意图状态（`explain_only` / `wrong` / `needs_review` / `bookmarked`）。**Mistake 表是否写入以 P0 真实错题为准**（`user_answer` ≠ `correct_answer`），与工作流意图关系如下：

| 工作流 `review_status` | 满足 P0 时创建 Mistake | 初始 `review_status` | 说明 |
|------------------------|------------------------|------------------------|------|
| `explain_only` | 否 | — | 不持久化 |
| `wrong` | 是 | `new` | 标准错题（须 `error_type` + `error_reason`） |
| `needs_review` | 否 | — | 做对不确信**不**入库；须答错才入库 |
| `bookmarked` | 否 | — | 收藏**不**入库；写 Bookmark + History |

工作流意图（做对不确信、收藏等）**不**写入 Mistake；作答对错与争议写入 **QuestionHistory**（`result`、`answer_disputed`），收藏写入 **Bookmark**。

---

## 5. Future Extension

以下能力**不在核心表强制实现**，通过扩展字段、关联表或 JSON `metadata` 预留。

### 5.1 AI 生成类似题

| 预留 | 说明 |
|------|------|
| `similar_question_ids` | 由本 Mistake 触发生成的仿题 ID 列表 |
| `drill_session_id` | 关联一次「错题巩固专项」练习会话 |
| `generation_prompt_version` | 仿题生成所用 Prompt / 模型版本，便于回归与质量追踪 |

**用途**：根据 `error_type` / `wrong_type` + `knowledge_point` 自动生成同类变式题，巩固薄弱点。

### 5.2 学习计划

| 预留 | 说明 |
|------|------|
| `study_plan_item_id` | 关联 `study_plan` 中的具体任务项 |
| `scheduled_review_at` | 下次间隔复习推荐时间（间隔重复算法 SM-2 等） |
| `priority_score` | 复习优先级分数，由错误频率、ECO 权重、考试日期综合计算 |

**用途**：错题库驱动每日「今日待复习」与周计划排期。

### 5.3 视频课程关联

| 预留 | 说明 |
|------|------|
| `related_lesson_ids` | 关联教学视频 / 微课章节 ID |
| `related_lesson_timestamps` | 视频内跳转时间点（如 `{ "lesson_id": "...", "start_sec": 120 }`） |

**用途**：错题讲解页一键跳转至对应课程片段。

### 5.4 用户画像

| 预留 | 说明 |
|------|------|
| `wrong_type_secondary` | 次要错因类型 |
| `user_mistake_pattern_tags` | 与全局错误模式匹配的标签（如「总把 First 当 Best」） |
| `exam_target_date` | 快照用户备考目标日期（用于优先级，也可仅存于 User 表） |
| `locale` | 用户语言偏好，影响讲解与术语展示 |

**用途**：跨题聚合为「个人薄弱画像」，供 Agent 个性化教练话术使用。

### 5.5 多版本 PMP 考试支持

| 预留 | 说明 |
|------|------|
| `exam_version` | 考纲版本（如 `ECO_2021`、`ECO_2025`） |
| `pmbok_version` | 参考 PMBOK 版本（如 `6`、`7`） |
| `knowledge_framework` | 知识点挂载体系：`pmbok_process` / `agile_practice` / `eco_task` |

**用途**：考纲更新后，旧错题仍可保留历史标签，新题按新版本标注，统计时可按版本过滤。

### 5.6 其他通用扩展

| 预留 | 说明 |
|------|------|
| `metadata` | 开放 JSON，承载尚未升维为列的实验性字段 |

> **已废弃（勿写入 `mistake_memory.json`）**：`ingestion_tag`、`is_correct`、`answer_disputed` — 分别由 Bookmark、QuestionHistory `result`、QuestionHistory `answer_disputed` 承担。

---

## 附录：最小 JSON 示例

```json
{
  "mistake_id": "550e8400-e29b-41d4-a716-446655440000",
  "user_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
  "question_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "source": "某某培训机构模拟题",
  "exam_set": "模拟卷第 2 套 Q15",
  "question_text": "项目执行期间，两名团队成员因工作分配发生冲突……",
  "options": {
    "A": "立即将冲突上报给发起人",
    "B": "与双方会面，合作解决问题",
    "C": "重新分配任务，避免进一步冲突",
    "D": "更新资源管理计划"
  },
  "user_answer": "A",
  "correct_answer": "B",
  "knowledge_point": ["冲突管理", "合作/解决问题"],
  "knowledge_points": ["冲突管理", "合作/解决问题"],
  "answer_explanation": {
    "correct_answer_reason": "冲突应首先通过面对面沟通与合作解决",
    "memory_rule": "团队冲突：先面对面，再升级"
  },
  "project_type": "predictive",
  "project_phase": "executing",
  "exam_domain": "people",
  "process": "管理团队",
  "error_type": "scenario_judgment_error",
  "error_reason": "用户选择了直接上报，跳过了与团队成员的协作沟通，违反 Collaborate before escalate 原则。",
  "wrong_type": "scenario_judgment_error",
  "wrong_reason": "用户选择了直接上报，跳过了与团队成员的协作沟通，违反 Collaborate before escalate 原则。",
  "difficulty": "medium",
  "confidence_level": "high",
  "review_status": "new",
  "review_count": 0,
  "created_at": "2026-07-28T11:00:00+08:00",
  "updated_at": "2026-07-28T11:00:00+08:00"
}
```
