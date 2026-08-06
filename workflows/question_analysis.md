# Question Analysis Workflow

> 题目分析工作流：用户提交 PMP 题目（文字、截图或解析）后，Agent 完成识别、讲解、错因判断与结构化归档。
>
> **输出职责**：本文件 §7 为题目分析的**唯一固定输出模板**；`skill.md` 仅保留全局输出规则，不重复本模板。
>
> 本文件定义**执行步骤与输出契约**，不替代 `skill.md` 中的角色定义与推理原则。
>
> **前置**：用户单题录入（文字 / 截图 / 补答案）先执行 `workflows/question_capture.md`，将 `CAPTURE_RECORD` 作为本章 §2 的预填充输入。

---

## 0. 执行原则

### 事实与推测分离

所有输出必须区分两类信息：

| 标记 | 含义 | 示例 |
|------|------|------|
| **【事实】** | 用户原文、截图 OCR 结果、用户明确提供的答案或解析 | 题干原文、用户说「我选了 B」 |
| **【推测】** | Agent 基于 PMP 知识推断，但用户未提供 | 推断正确答案、推断项目阶段 |
| **【待确认】** | 信息缺失，无法可靠判断 | 题目来源未知、套卷编号不明 |

**硬性约束：**

- 不得凭空补全题干、选项、用户答案或正确答案
- 截图 OCR 结果不完整时，只分析已识别部分，并列出缺失字段
- 无用户答案时，**不得**擅自判断用户错因
- 无官方解析时，**不得**冒充「官方解析」，只能给出基于 PMP 框架的「考试逻辑分析」

### 兼容性说明

本工作流设计为平台无关，可被以下场景调用：

- **对话式 Agent**：用户直接粘贴或上传
- **截图 OCR 管道**：上游传入 `ocr_text` + `image_ref`，本工作流负责结构化与讲解
- **数据库 / Web 产品**：下游读取 `Data Handoff` 区块的 JSON 字段入库

---

## 1. Trigger（触发条件）

识别用户意图，进入本工作流。以下任一条件满足即触发：

| 触发场景 | 识别信号 | 默认行为 |
|----------|----------|----------|
| 用户上传题目截图 | 图片附件，或提及「看图」「这道题」 | 尝试 OCR → 结构化 → 讲解；缺失信息标【待确认】 |
| 用户粘贴题目文字 | 含题干 + 选项的纯文本 | 直接结构化 → 讲解 |
| 用户提供题目 + 自己的答案 + 正确答案 | 明确给出 A/B/C/D 或选项内容 | 结构化 → 讲解 → 错因判断 → 按规则归档 |
| 用户仅询问题目，不要求持久化 | 「帮我讲讲」「分析一下」「不用保存」 | 完整讲解，`review_status = explain_only`；有作答仍写 History |
| 用户说「加入错题本」等 | 在**已答错**场景下等同于确认复习关注；**不**单独触发入库（答错已自动写 Mistake） | 讲解 + 若 P0 答错则已入库；答对则说明「本题未答错，不入错题库」 |

### 意图消歧

若触发信号冲突，按以下优先级处理：

1. **用户显式指令**（「不要保存」「只要讲解」）> 默认归档行为（`force_skip` 时不写 Mistake）
2. **「收藏这题」** → 仅 Bookmark；**不**写 Mistake
3. **无显式指令**：有 `user_answer` → **必写 History**；若答错 → **自动写 Mistake**（**禁止**询问「是否保存错题」）

---

## 2. Input Extraction（输入提取）

从用户消息、截图 OCR 结果、附件元数据中，提取并区分以下字段。

### 2.1 字段清单

| 字段 | 说明 | 提取来源 |
|------|------|----------|
| `question_text` | 题干全文（含场景描述） | 用户粘贴 / OCR |
| `options` | 选项列表，格式 `{"A": "...", "B": "...", ...}` | 用户粘贴 / OCR |
| `user_answer` | 用户选择的选项或自述思路 | 用户明确提供；否则 `null` + 【待确认】 |
| `correct_answer` | 正确答案选项 | 用户提供 / 官方解析 / 【推测】标注 |
| `official_explanation` | 教材、题库、解析页原文 | 用户粘贴；无则 `null` |
| `source` | 题目来源（PMBOK、某题库、真题、自编等） | 用户说明 / 图片水印 / 【待确认】 |
| `exam_set` | 套卷名称或编号（如「第 3 套模拟题 Q12」） | 用户说明 / 【待确认】 |
| `attempt_date` | 做题日期 | 用户说明 / 消息时间戳 / 【待确认】 |

### 2.2 提取步骤

```
Step 1: 获取原始输入
  ├─ 文字 → 直接进入 Step 2
  └─ 截图 → OCR（或读取上游 ocr_text）→ 进入 Step 2

Step 2: 切分结构
  ├─ 分离题干与选项（A/B/C/D 或 1/2/3/4）
  ├─ 识别是否夹杂「我的答案」「正确答案」「解析」段落
  └─ 保留原文，不做改写

Step 3: 标注置信度
  ├─ 高置信：原文清晰、选项完整
  ├─ 低置信：OCR 乱码、选项缺失、图片模糊
  └─ 低置信字段 → 列出【待确认】清单，向用户确认

Step 4: 去重检测（若有历史题库访问能力）
  ├─ 题干相似度 / 选项指纹匹配
  └─ 命中已有记录 → 标记 duplicate_of，走更新逻辑（见 §6）
```

### 2.3 提取输出示例

```markdown
### 提取结果

| 字段 | 值 | 置信度 |
|------|-----|--------|
| 题干 | 【事实】... | 高 |
| 选项 A–D | 【事实】... | 高 |
| 用户答案 | 【待确认】用户未提供 | — |
| 正确答案 | 【推测】B（依据 PMP 决策逻辑） | 中 |
| 官方解析 | null | — |
| 来源 | 【待确认】 | — |
| 套卷 | 【待确认】 | — |
| 做题日期 | 【事实】2026-07-28（消息时间） | 中 |
```

---

## 3. PMP Analysis Framework（分析框架）

按以下顺序执行判断，**每一步都必须显式输出**，不可跳步。与 `skill.md` §4 保持一致并扩展。

### Step 1: Project Type（项目类型）

| 类型 | 判断信号 |
|------|----------|
| **Predictive** | 变更控制委员会、WBS、甘特图、基准、瀑布里程碑 |
| **Agile** | Sprint、Product Owner、Scrum Master、迭代、用户故事、看板 |
| **Hybrid** | 同时出现预测型计划与敏捷交付元素 |

- 无法判断 → 标注【待确认】，并说明缺失的场景线索

### Step 2: Project Phase（项目阶段）

| 阶段 | 典型信号 |
|------|----------|
| **Initiating** | 章程、商业论证、识别相关方 |
| **Planning** | 范围、进度、成本、风险、质量计划 |
| **Executing** | 指导团队、管理沟通、实施交付 |
| **Monitoring and Controlling** | 偏差分析、变更请求、绩效审查、问题升级 |
| **Closing** | 收尾、经验教训、最终报告、移交 |

- 一道题可能涉及多个阶段，标注**主要阶段**与**次要阶段**

### Step 3: ECO Domain（考试大纲领域）

| 领域 | 覆盖范围 |
|------|----------|
| **People** | 团队、冲突、领导力、相关方、沟通 |
| **Process** | 范围、进度、成本、质量、风险、采购、整合等 |
| **Business Environment** | 合规、组织战略、价值交付、治理 |

### Step 4: Knowledge Area & Specific Points（知识领域与具体知识点）

- 映射到 PMBOK 过程 / Agile 实践 / 考试大纲条目
- 列出 1–3 个**核心知识点**（应考级别，非教材目录堆砌）
- 示例：`实施整体变更控制`、`冲突管理 - 合作/解决问题`、`敏捷 - 仆人式领导力`

### Step 5: Question Intent（题目问法）

识别题干关键词，决定答题策略：

| 问法 | 含义 | 答题侧重 |
|------|------|----------|
| **First** | 当前情境下**最先**应采取的行动 | 第一步，非最终方案 |
| **Next** | **下一步**做什么 | 已完成动作的后续 |
| **Best** | **最佳**做法 | 综合最优，可能非第一步 |
| **Most Appropriate** | **最合适**的做法 | 情境适配，排除过度/不足反应 |
| **Should Do** | **应该**怎么做 | 规范/流程要求的行为 |

- 若题干无明确问法词，标注为「隐含意图」并说明推断依据

### Step 6: Scenario Keywords & Trap Analysis（场景词与干扰项陷阱）

**场景关键词：** 提取影响决策的约束词（如：客户不满、范围蔓延、资源冲突、监管要求、紧急、全球团队、虚拟团队）。

**干扰项陷阱（常见类型）：**

| 陷阱类型 | 说明 |
|----------|------|
| 跳过分析直接行动 | 未评估就执行变更或上报 |
| 过度升级 | 未经团队/相关方沟通即上报管理层 |
| 混淆 First 与 Best | 把长期最优解当作第一步 |
| 角色越权 | PM 做了应由 PO / SM / 发起人的事 |
| 流程顺序颠倒 | 先执行后批准、先关闭后验收 |
| 术语近义混淆 | 风险 vs 问题、章程 vs 计划、缺陷 vs 变更请求 |
| 绝对化措辞 | 「立即」「必须」「总是」的极端选项 |

---

## 4. Answer Explanation（答案讲解）

无论用户是否要求保存，讲解部分**必须完整输出**以下内容。

### 4.1 必答项

1. **正确答案** — 明确给出选项字母及内容（【事实】或【推测】标注）
2. **正确答案为什么成立** — 结合 §3 框架：类型 → 阶段 → 领域 → 问法 → PMP 决策逻辑
3. **每个错误选项为什么不优先** — 逐项分析，说明「为何不选」而非仅说「错误」
4. **标准处理顺序** — 若题目涉及多步流程，给出正确先后顺序（可用编号列表）
5. **可迁移判断方法** — 本题提炼 1 条通用规则，可应用于同类题

### 4.2 PMP 决策逻辑（讲解时必须引用）

按优先级应用（与 `skill.md` 一致）：

1. **Analysis before action** — 先分析，再行动
2. **Collaborate before escalate** — 先协作沟通，再升级上报
3. **Follow process before changing** — 先遵循流程，再考虑变更
4. **Update plan before execution** — 先更新计划，再执行

### 4.3 讲解约束

- 有官方解析时：先引用【事实】原文要点，再补充考试逻辑
- 无官方解析时：全部标注【推测】，并说明推断链
- 正确答案不确定时：**不得**给出肯定讲解；列出候选答案及各自成立条件，请用户确认

---

## 5. Mistake Classification（错因分类）

### 5.1 分类枚举

用户做错时（或用户自述「我错了」），**必须且只能**从以下类别中选择主类型（可选一个次要类型）：

| 类型 | 英文 | 适用情形 |
|------|------|----------|
| 知识盲区 | **Knowledge Gap** | 不了解该知识点或过程 |
| 概念混淆 | **Concept Confusion** | 两个相近概念选错 |
| 场景判断错误 | **Scenario Judgment Error** | 误解题干情境、约束或相关方诉求 |
| 流程顺序错误 | **Process Sequence Error** | 知道概念但步骤/先后顺序错误 |
| 角色与职责错误 | **Role and Responsibility Error** | 混淆 PM / PO / SM / 发起人 / 团队成员职责 |
| 术语理解问题 | **Terminology Problem** | 英文关键词或专业术语理解偏差 |
| 粗心 | **Carelessness** | 看错题干、选错选项、忽略否定词 |
| 信息不足 | **Insufficient Information** | 题干/选项信息不够，用户或 Agent 均无法可靠判断 |

### 5.2 分类规则

| 条件 | 行为 |
|------|------|
| 用户**未提供**答案 | `error_type = null`；输出「用户错因：【待确认】未提供用户答案，无法判断错因」 |
| 用户答案**正确**（对 **adjudication_answer**） | `error_type = null`；可标注「掌握状态：已掌握 / 不确定」 |
| 用户答案**错误** | 给出主类型（写入 `error_type`）+ 判断依据（`error_reason`） |
| 用户自述思路但未选选项 | 按自述思路判断，标注【推测】 |

### 5.3 输出格式

```markdown
## 用户错因

- **错因类型**：Scenario Judgment Error（主）/ Process Sequence Error（次）
- **判断依据**：【事实】用户选了 A；该选项跳过了与相关方沟通直接上报...
- **与历史模式关联**（若有记录）：你在「冲突管理」类题目中已 3 次出现同类错误...
```

---

## 5.5 Answer Evaluation（答案可信层 · v0.1.1）

在 **§4 讲解完成** 之后、**§6 分流** 之前执行。Coach **不是题库答案搬运工具**；须独立给出考试逻辑判断。

### 5.5.1 输入

| 字段 | 来源 |
|------|------|
| `platform_answer` | Capture / 用户粘贴 / OCR 解析页标答（映射采集侧「标答」） |
| `user_answer` | 用户作答 |
| `coach_answer` | 本工作流 §3–§4 推理后的应选项 |
| `coach_answer_reason` | 一句话决策链（写入 `explanation.correct_answer_reason` 或同等） |

### 5.5.2 产出 `answer_evaluation`

| 字段 | 说明 |
|------|------|
| `platform_answer` | 可 `null` |
| `coach_answer` | 可 `null`（则 `answer_status=uncertain`） |
| `coach_answer_reason` | `coach_answer` 非空时必填 |
| `answer_confidence` | `high` / `medium` / `low`（针对 Coach） |
| `answer_status` | `coach_only` / `aligned` / `disputed` / `uncertain` |
| `answer_disputed` | `true` 当且仅当 `answer_status=disputed` |
| `adjudication_answer` | 按 `decision_rules.md` §0.1 |
| `ael_version` | `"0.1.1"` |

### 5.5.3 错因与对错

- **对错**：`user_answer` vs **`adjudication_answer`**（非单独 vs `platform_answer`）。  
- **错因**（§5）：仅当相对 `adjudication_answer` 答错时填写 `error_type` / `error_reason`。  
- **争议且用户 = Coach**：`error_type=null`；History `result=correct`，`answer_disputed=true`。

---

## 6. Data Routing（三分流 · 自动）

根据作答结果 **自动分流**，**禁止**询问「是否保存错题」。

### 6.1 决策矩阵

| 场景 | History | Mistake | Bookmark | 用户提示 |
|------|:-------:|:-------:|:--------:|----------|
| 仅讲解、无作答 | 可选 | 否 | 否 | — |
| 答对（对 **adjudication**） | ✅ | 否 | 仅主动收藏 | — |
| **答错**（对 **adjudication**） | ✅ | **✅ 自动**（`decision_rules` §0–§1） | 仅主动收藏 | 「本题已自动加入错题库」 |
| **争议**：用户=Coach ≠ 平台 | ✅ `correct` + `answer_disputed` | **否** | — | 说明未按题库记错 |
| **争议**：用户≠Coach（Coach 可信） | ✅ `wrong` | ✅ | — | 同答错 |
| `uncertain` / 无 adjudication | ✅ `unknown` | 否 | — | 建议补信息或确认标答 |
| 用户说「收藏这题」等 | 不变 | 不变 | ✅ | 「已加入收藏」 |
| 与已有错题重复再次答错 | ✅ 新 History | 更新 Mistake | — | 「错题记录已更新」 |

### 6.2 重复错题处理

1. 保留原 `question_id` / `mistake_id`
2. 追加 History；Mistake：`repeated_count += 1`，刷新 `error_type`、`error_reason`（**禁止**新写 `mistake_type` / `eco_domain`）
3. 输出：「本题已存在于错题本，已更新做题记录」

### 6.3 置信度说明

- Coach 侧 `answer_confidence=low` 或 `answer_status=uncertain`：**不写 Mistake**（仅 History）。  
- `answer_confidence` 为 medium/high 且相对 adjudication 答错：可入库；`low` 时 Mistake 标 `confidence_level=low` 并提示用户。  
- 题库标答仅为【事实·来源】；讲解 §2 须区分 **Coach 判断** 与 **平台标答**（见 §7 可选「答案对照」）。

**删除**：任何「回复保存错题再入库」的交互。

---

## 7. Fixed Output（固定输出）

每次题目分析**必须**按以下结构输出，便于用户阅读与下游解析。

```markdown
# PMP 题目分析

## 1. 题目信息

- **题干**：[原文或结构化摘要]
- **选项**：A ... / B ... / C ... / D ...
- **用户答案**：[选项 / 未提供 / 待确认]
- **来源**：[事实 / 待确认]
- **套卷**：[事实 / 待确认]
- **做题日期**：[事实 / 待确认]
- **信息置信度**：[高 / 中 / 低] + 缺失项说明

## 2. 正确答案

**[选项字母]** — [一句话结论]（**Coach 考试逻辑判断**）

> 标注：【事实】用户提供 / 【推测】Agent 推断

### 2b. 答案对照（仅当 `answer_disputed` 或 `platform_answer` ≠ `coach_answer`）

| 来源 | 选项 | 说明 |
|------|------|------|
| 题库/App | … | 【事实】 |
| Coach（考试逻辑） | … | 置信度：高/中/低 |
| 本次判题基准 | … | `adjudication_answer` |

## 3. 项目类型与阶段

- **Project Type**：Predictive / Agile / Hybrid
- **Project Phase**：Initiating / Planning / Executing / Monitoring and Controlling / Closing
- **主要依据**：[场景关键词]

## 4. 考察知识点

- **ECO Domain**：People / Process / Business Environment
- **知识领域**：[如 整合管理、风险管理]
- **具体知识点**：
  1. ...
  2. ...

## 5. PMP 考试逻辑

- **题目问法**：First / Next / Best / Most Appropriate / Should Do
- **决策链**：项目类型 → 阶段 → 领域 → 问法 → 决策原则
- **关键场景词**：...
- **标准处理顺序**：
  1. ...
  2. ...
  3. ...

## 6. 选项逐项分析

### A — [选项摘要]
- **结论**：正确 / 不优先 / 待确认
- **分析**：...

### B — [选项摘要]
- **结论**：...
- **分析**：...

（C、D 同理）

### 干扰项陷阱总结
- ...

## 7. 用户错因

- **错因类型**：[分类名 / 未提供答案，无法判断 / 答对无需错因分析]
- **判断依据**：...

## 8. 一句话记忆规则

> [可迁移到同类题的一句话口诀或判断规则]

## 9. 后续学习建议

1. **立即复习**：...
2. **专项练习**：...
3. **防再错检查清单**：...

## 10. 归档状态

- **History**：已写入 / 跳过
- **Mistake**：答错则 **已自动加入错题库**（含错误类型 / 原因 / 知识点）
- **Bookmark**：仅当用户主动收藏 → 已加入 / 未收藏
- **待确认项**：[列出需用户补充的字段]
```

> 原 §10「是否保存 / review_status=bookmarked」已废弃；收藏与错题分离。

---

## 8. Data Handoff（数据交接）

分析完成后，在输出末尾生成一份**结构化 JSON**，供数据库或 Web 产品消费。

> 本节只定义交接字段与取值约定，**不定义**完整数据库 Schema。字段类型、索引、关联表见 `database/` 目录。

### 8.1 交接块格式

在 Markdown 输出最后附上：

````markdown
<!-- DATA_HANDOFF:BEGIN -->
```json
{ ... }
```
<!-- DATA_HANDOFF:END -->
````

- 机器解析：提取 `DATA_HANDOFF` 注释块中的 JSON
- 字段值为 `null` 表示未知；不得用空字符串代替缺失

### 8.2 字段定义

| 字段 | 类型 | 说明 | 取值约定 |
|------|------|------|----------|
| `question_text` | string | 题干全文 | 【事实】原文 |
| `options` | object | 选项键值对 | `{"A":"...","B":"...","C":"...","D":"..."}` |
| `user_answer` | string \| null | 用户答案 | 单选 `"B"`；多选 `["A","C"]`；未提供 `null` |
| `correct_answer` | string \| null | **判题基准**（= `adjudication_answer`） | 与 `answer_evaluation.adjudication_answer` 一致 |
| `source` | string \| null | 题目来源 | 未知 `null` |
| `exam_set` | string \| null | 套卷名称或编号 | 未知 `null` |
| `project_approach` | string | 项目类型 | `predictive` / `agile` / `hybrid` / `unknown` |
| `project_phase` | string | 主要项目阶段 | `initiating` / `planning` / `executing` / `monitoring_controlling` / `closing` / `unknown` |
| `exam_domain` | string | ECO 领域（对外字段） | `people` / `process` / `business_environment` / `unknown` |
| `knowledge_points` | string[] | 具体知识点列表 | 1–5 条，短标签 |
| `error_type` | string \| null | 错因分类 | 枚举见 §5.1 小写蛇形；无错因 `null` |
| `error_reason` | string \| null | 错因说明 | 答错必填 |
| `explanation` | object | 讲解摘要 | 见 §8.3 |
| `review_status` | string | 复习/归档状态 | `explain_only` / `wrong` / `needs_review` / `bookmarked` |
| `created_at` | string | 记录创建时间 | ISO 8601，如 `2026-07-28T10:40:00+08:00` |

### 8.3 `explanation` 对象结构

```json
{
  "correct_answer_reason": "正确答案成立原因（简短）",
  "pmp_logic_summary": "考试逻辑一句话摘要",
  "option_analysis": {
    "A": "不优先原因",
    "B": "正确",
    "C": "不优先原因",
    "D": "不优先原因"
  },
  "standard_sequence": ["步骤1", "步骤2", "步骤3"],
  "memory_rule": "一句话记忆规则",
  "trap_patterns": ["陷阱1", "陷阱2"]
}
```

### 8.4 可选扩展字段

以下字段可在有数据时附加，不强制：

| 字段 | 说明 |
|------|------|
| `answer_evaluation` | **v0.1.1** 对象：`platform_answer`、`coach_answer`、`coach_answer_reason`、`answer_confidence`、`answer_status`、`answer_disputed`、`adjudication_answer`、`ael_version` |
| `correct_answer_confidence` | 同 `answer_evaluation.answer_confidence`（兼容） |
| `official_explanation` | 官方解析原文 |
| `attempt_date` | 做题日期 ISO 8601 |
| `question_intent` | `first` / `next` / `best` / `most_appropriate` / `should_do` / `implicit` |
| `duplicate_of` | 已有题目 ID |
| `pending_confirmations` | 待用户确认字段名数组 |
| `ocr_metadata` | `{ "source": "ocr", "confidence": 0.85, "image_ref": "..." }` |

**历史只读别名**（禁止出现在新 DATA_HANDOFF）：`mistake_type`、`eco_domain`、`mistake_reason` — 读取旧数据时等同 `error_type` / `exam_domain` / `error_reason`（见 `database/schema_overview.md` §1.1）。

### 8.5 完整示例

```json
{
  "question_text": "项目执行期间，两名团队成员因工作分配发生冲突...",
  "options": {
    "A": "立即将冲突上报给发起人",
    "B": "与双方会面，合作解决问题",
    "C": "重新分配任务，避免进一步冲突",
    "D": "更新资源管理计划"
  },
  "user_answer": "A",
  "correct_answer": "B",
  "source": null,
  "exam_set": null,
  "project_approach": "predictive",
  "project_phase": "executing",
  "exam_domain": "people",
  "knowledge_points": ["冲突管理", "合作/解决问题", "团队领导力"],
  "error_type": "scenario_judgment_error",
  "error_reason": "用户选了 A，跳过协作沟通直接上报发起人",
  "explanation": {
    "correct_answer_reason": "冲突应首先通过面对面沟通与合作解决，而非直接升级",
    "pmp_logic_summary": "Collaborate before escalate — PM 应先促成双方协商",
    "option_analysis": {
      "A": "跳过分析与协作，过度升级",
      "B": "正确 — 符合合作/解决问题策略",
      "C": "未了解根因就调整分配，可能回避问题",
      "D": "与当前冲突无直接关联，非第一步"
    },
    "standard_sequence": [
      "了解冲突背景与双方诉求",
      "促成面对面沟通",
      "合作制定解决方案",
      "必要时再升级"
    ],
    "memory_rule": "团队冲突：先面对面，再升级；先合作，再强制",
    "trap_patterns": ["过度升级", "混淆 First 与 Best"]
  },
  "review_status": "wrong",
  "created_at": "2026-07-28T10:40:00+08:00",
  "correct_answer_confidence": "high",
  "answer_evaluation": {
    "platform_answer": "B",
    "coach_answer": "B",
    "coach_answer_reason": "冲突应首先通过面对面沟通与合作解决，而非直接升级",
    "answer_confidence": "high",
    "answer_status": "coach_only",
    "answer_disputed": false,
    "adjudication_answer": "B",
    "ael_version": "0.1.1"
  },
  "question_intent": "first",
  "pending_confirmations": ["source", "exam_set"]
}
```

---

## 9. 工作流总览

```
用户输入（文字 / 截图 / 解析）
        │
        ▼
  ┌─────────────┐
  │ 1. Trigger  │ 识别意图：讲解 / 保存 / 收藏
  └──────┬──────┘
         ▼
  ┌──────────────────┐
  │ 2. Input Extract │ 结构化字段 + 置信度 + 去重
  └──────┬───────────┘
         ▼
  ┌──────────────────────┐
  │ 3. PMP Framework     │ 类型→阶段→领域→知识点→问法→陷阱
  └──────┬───────────────┘
         ▼
  ┌──────────────────────┐
  │ 4. Answer Explanation│ 正确答案 + 逐项分析 + 迁移规则
  └──────┬───────────────┘
         ▼
  ┌──────────────────────┐
  │ 5. Mistake Classify  │ 有用户答案才判错因
  └──────┬───────────────┘
         ▼
  ┌──────────────────────┐
  │ 5.5 Answer Evaluation│ platform vs coach → adjudication
  └──────┬───────────────┘
         ▼
  ┌──────────────────────┐
  │ 6. Save Decision     │ review_status + 重复题更新
  └──────┬───────────────┘
         ▼
  ┌──────────────────────┐
  │ 7. Fixed Output      │ 固定 Markdown 结构
  └──────┬───────────────┘
         ▼
  ┌──────────────────────┐
  │ 8. Data Handoff      │ JSON 交接块 → DB / Web
  └──────────────────────┘
```

---

## 10. 与其他工作流的关系

| 下游 | 触发条件 |
|------|----------|
| `mistake_classification.md` | 多题错因聚合、模式识别（单题分类在本工作流 §5 完成） |
| `study_plan.md` | 错题积累后，根据 `knowledge_points` 与 `error_type` 调整学习计划 |
| `database/question_schema.md` | 入库时映射 `Data Handoff` 字段到持久化模型 |

单题分析的终点是 **§7 Fixed Output + §8 Data Handoff**；是否真正写入数据库，由调用方（Agent 工具 / Web API）执行。
