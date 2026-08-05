# Material Processing Workflow

> 可维护的 PMP 学习资料处理流水线：将 `source_materials/` 及用户上传的原始资料，转化为 Agent 可理解、可检索的结构化知识。
>
> **硬性规则**：`source_materials/` 原文不得被 Agent 直接引用；须经本流程写入 `knowledge/` 后方可调用。

---

## 1. Purpose

该流程负责将**原始资料**转换为 **Agent 可理解的结构化知识**。

加工目标不是「存档 PDF」，而是产出可供 `question_analysis`、决策引擎、学习计划等工作流调用的**判断型知识**——含定义、关键词、考试规则、示例与易错点。

### 输入

| 类型 | 典型形式 | 建议存放 |
|------|----------|----------|
| **PDF 教材** | PMBOK、辅导书、考纲 PDF | `source_materials/textbook/`、`source_materials/exam/` |
| **PPT 课程** | 培训讲义、冲刺幻灯片 | `source_materials/course/` |
| **视频笔记** | 字幕、文字稿、听课笔记 | `source_materials/course/` |
| **题库** | 套卷 PDF/Excel/Word、带解析题库 | `source_materials/question_bank/` |
| **用户上传资料** | 聊天附件、截图、个人笔记 | 登记 `source_path` + `user_id`（若适用） |

另支持：Word、Excel、网页剪藏、截图（见 §3 Step 1）。

### 输出

每条加工结果应落入以下五类产出（可组合）：

| 产出类型 | 说明 | 典型落点 |
|----------|------|----------|
| **知识点** | 概念、过程、定义、ITTO 要点 | `knowledge/pmbok/`、`knowledge/agile/` |
| **关键词** | 题干信号词、英文术语、问法词 | `knowledge/language/`、`knowledge/exam/` |
| **考试规则** | 决策顺序、口诀、优先级原则 | `knowledge/decision_framework/`、`knowledge/exam/` |
| **示例** | 场景案例、例题片段、对比表 | 嵌入知识点条目或 `examples/` |
| **易错点** | 易混概念、干扰项套路、陷阱模式 | `knowledge/language/confusing_terms.md`、`knowledge/exam/trap_patterns.md` |

机器可读交接：`MATERIAL_HANDOFF` JSON + 加工报告（见 §4、§5）。

---

## 2. Source Classification

在提取内容前，先判定**来源类型**（`source_class`）与**内容类型**（`content_type`）。二者可组合。

### 2.1 来源分类（Source Class）

| 类别 | 标识 | 典型示例 | 信任度 | 默认落点 |
|------|------|----------|--------|----------|
| **A. Official Source** | `official` | PMBOK、ECO、PMI 官方说明 | 高 | `knowledge/pmbok/`、`knowledge/exam/` |
| **B. Training Material** | `training` | 培训讲义、视频笔记、讲师口诀 | 中 | `knowledge/decision_framework/`、`knowledge/course/`（若扩展） |
| **C. Question Bank** | `question_bank` | 模拟题、真题合集、带解析题库 | 中（答案以原文为准） | `examples/questions/` |
| **D. User Data** | `user_data` | 个人错题、用户笔记、截图错题 | 视用户输入 | `memory/`、`database/`（经 `question_analysis`） |

### 2.2 内容类型（Content Type）

| content_type | 识别信号 | 与 source_class 常见组合 |
|--------------|----------|--------------------------|
| `knowledge` | 概念、过程、章节、ITTO | A + B |
| `terminology` | 英中词汇、缩写表 | A + B |
| `exam_rule` | 考纲权重、题型说明、答题策略 | A |
| `question` | 题干 + 选项（≥2） | C、D |
| `mixed` | 同一文件含多类 | 拆分为多个 Processing Unit |

### 2.3 分类决策树

```
用户上传 / source_materials/
    │
    ├─ 含题干+选项？ ──是──→ C 或 D（有 user_answer → D）
    │
    ├─ 含 ECO / PMBOK 官方标识？ ──→ A
    │
    ├─ 含讲师/课程/口诀？ ──→ B
    │
    └─ 英中词汇表？ ──→ terminology → knowledge/language/
```

### 2.4 与 `source_materials/` 目录映射

| 目录 | 推荐 source_class |
|------|-------------------|
| `exam/` | A |
| `textbook/` | A |
| `course/` | B |
| `terminology/` | A / B |
| `question_bank/` | C |

---

## 3. Processing Pipeline

**按顺序执行，不可跳步。** 任一步失败则挂起或输出 Need Review，不静默入库。

```
Raw Material（原始资料）
        │
        ▼
Extract Text（提取文本）
        │
        ▼
Clean Content（清洗内容）
        │
        ▼
Classify Topic（分类主题 / 来源）
        │
        ▼
Map Knowledge Point（映射知识点）
        │
        ▼
Generate Structured Knowledge（生成结构化知识）
        │
        ▼
Store（写入 knowledge / examples / memory）
```

### Step 1 — Raw Material（原始资料）

| 动作 | 说明 |
|------|------|
| 接收 | 文件或文本 + `source_path` + `source_label` |
| 登记 | 分配 `processing_id`（UUID） |
| 保留原件 | **不修改、不删除** `source_materials/` 内文件 |

### Step 2 — Extract Text（提取文本）

| 输入类型 | 提取方式 |
|----------|----------|
| PDF | 文本层；扫描件 → OCR（记录 `ocr_confidence`） |
| PPT | 幻灯片标题 + 要点 + 演讲者备注 |
| 视频笔记 | 去时间戳，按段落/章节切分 |
| 题库 | 按题号/行切分；保留选项结构 |
| 截图 | OCR → `ocr_text` |

输出：`raw_units[]`（`text`、`page_range`、`structure_hints`）。

### Step 3 — Clean Content（清洗内容）

| 动作 | 说明 |
|------|------|
| 去噪 | 页眉页脚、页码、重复水印、广告 |
| 规范化 | UTF-8；统一换行；保留标题层级 |
| 标注 | 【事实】原文摘录 vs 待加工片段 |
| 拒绝 | 加密/损坏/整页 OCR 不可读 → Need Review |

**禁止**：将清洗结果当作最终知识入库（仍为中间态）。

### Step 4 — Classify Topic（分类主题）

应用 §2：输出 `source_class`、`content_type`、`primary_topic`（如 scope / risk / agile / terminology）。

大文件按章/套卷拆为多个 `Processing Unit`，分别分类。

### Step 5 — Map Knowledge Point（映射知识点）

| 动作 | 说明 |
|------|------|
| 对齐考纲 | ECO Domain：People / Process / Business Environment |
| 对齐 PMBOK | 知识领域、过程名（如 Validate Scope） |
| 对齐已有库 | 查 `knowledge/terminology/pmp_glossary.md`、`knowledge/language/synonym_mapping.md` |
| 去重指纹 | `concept_id` = 过程名 + 知识域名；`question_id` = 题干 hash |
| 标记关系 | `related_concepts[]`、`confusing_terms[]` |

无法映射 → `knowledge_point: null` + Need Review，**不编造**标准过程名。

### Step 6 — Generate Structured Knowledge（生成结构化知识）

按 §4（知识点）或 §5（题库）模板生成 Markdown + JSON 字段。

必须区分：

- **【事实】**：原文明确存在
- **【推测】**：Agent 补充的考试语境（标 `confidence`）
- **Need Review**：无法确认

### Step 7 — Store（存储）

| source_class | 目标路径 |
|--------------|----------|
| A 官方 / 教材 | `knowledge/pmbok/`、`knowledge/exam/`、`knowledge/agile/` |
| B 培训 | `knowledge/decision_framework/` |
| 术语 | `knowledge/language/`、`knowledge/terminology/` |
| C 题库 | `examples/questions/` |
| D 用户错题 | `memory/mistake_memory` 或 `database/`（经分析工作流） |

索引：追加 `knowledge/source/imported_materials.md`（或 `knowledge/terminology/import_log.md`）。

输出：§4 加工报告 + `MATERIAL_HANDOFF` JSON。

---

## 4. Knowledge Extraction Format

每个知识点**必须**输出以下字段。缺失时置 `null` 并说明原因，**不得虚构**。

| 字段 | 英文键 | 必须 | 说明 |
|------|--------|------|------|
| **Knowledge Name** | `knowledge_name` | 是 | 中文知识点名 |
| **English Term** | `english_term` | 推荐 | 英文原词/过程名；核心概念不可省略 |
| **Definition** | `definition` | 是 | 定义；【事实】摘自原文或【推测】标注 |
| **Related Process** | `related_process` | 推荐 | PMBOK 过程或敏捷实践 |
| **Exam Keywords** | `exam_keywords[]` | 推荐 | 题干信号词、缩写 |
| **Scenario** | `scenario` | 推荐 | 典型考题情境 |
| **Common Trap** | `common_trap` | 推荐 | 易错点、干扰项套路 |
| **Memory Method** | `memory_method` | 否 | 口诀、对比、一句话规则 |

### 4.1 Markdown 模板

```markdown
---
id: {concept_id}
source_class: official | training
source: {source_label}
source_path: source_materials/...
pmbok_version: {见 §6}
exam_version: {见 §6}
confidence: high | medium | low
target_path: knowledge/pmbok/scope.md
---

## {Knowledge Name}（{English Term}）

### Definition
【事实|推测】{definition}

### Related Process
{related_process}

### Exam Keywords
- {keyword_1}
- {keyword_2}

### Scenario
> {scenario}

### Common Trap
{common_trap}

### Memory Method
> {memory_method}

### Source
- 页码/段落：{source_page}
```

### 4.2 产出类型映射

| 加工侧重 | 额外写入 |
|----------|----------|
| 关键词为主 | `exam_keywords` → 同步 `knowledge/language/keyword_mapping.md` |
| 考试规则为主 | `memory_method` + 决策顺序 → `knowledge/decision_framework/` |
| 易错点为主 | `common_trap` → `knowledge/language/confusing_terms.md` |
| 示例为主 | `scenario` → 嵌入条目或 `examples/` |

### 4.3 MATERIAL_HANDOFF（单条知识点 JSON）

```json
{
  "item_type": "knowledge_point",
  "knowledge_name": "确认范围",
  "english_term": "Validate Scope",
  "definition": "【事实】…",
  "related_process": "Validate Scope",
  "exam_keywords": ["Customer Acceptance", "Deliverable"],
  "scenario": "客户正式验收可交付成果",
  "common_trap": "与 Control Quality 混淆",
  "memory_method": "客户点头验收范围，检查合格是质量",
  "source_class": "official",
  "concept_id": "scope_validate_scope",
  "need_review": false
}
```

---

## 5. Question Bank Processing

`source_class = question_bank` 或用户错题（`user_data`）走本题库分支。

### 5.1 必提取字段

| 字段 | 英文键 | 必须 | 说明 |
|------|--------|------|------|
| **Question** | `question` | 是 | 题干原文 |
| **Options** | `options` | 是 | `{"A":"...","B":"..."}` |
| **Answer** | `correct_answer` | 否 | 原文有则【事实】；无则 `null` + Need Review |
| **Explanation** | `explanation` | 否 | 官方/讲义解析原文 |
| **Knowledge Point** | `knowledge_points[]` | 推荐 | 关联概念/过程 |
| **Difficulty** | `difficulty` | 否 | `easy` / `medium` / `hard` |
| **Trap Type** | `trap_type` | 推荐 | 对齐 `trap_patterns.md`（如 T02 过早升级） |

用户错题（D 类）额外字段：`user_answer`、`error_type` → 交 `question_analysis` 工作流。

### 5.2 处理流程

```
题库 Raw Material
    │
    ▼
按题切分 → Extract Question / Options
    │
    ▼
有解析？ → 提取 Answer / Explanation【事实】
无答案？ → correct_answer = null，Need Review
    │
    ▼
Map Knowledge Point + Trap Type【推测】须标注
    │
    ▼
Store → examples/questions/{source}_{set}.md
可选 → 触发 question_analysis 生成讲解
```

### 5.3 题库 Markdown 模板

```markdown
---
id: {question_id}
type: question_bank
source: {source_label}
exam_set: {set_name}
difficulty: medium
---

## Question
{question_text}

## Options
- A. …

## Answer
【事实】B

## Explanation
【事实】…

## Knowledge Points
- Validate Scope

## Trap Type
- Concept Confusion: Validate Scope vs Control Quality
```

---

## 6. Version Management

每条入库记录**必须**携带版本元数据，支持考纲更新与多版本共存。

| 字段 | 英文键 | 必须 | 说明 |
|------|--------|------|------|
| **PMBOK 版本** | `pmbok_version` | 推荐 | 如 `6`、`7`、`unknown` |
| **更新时间** | `updated_at` | 是 | 本次加工时间 ISO 8601 |
| **来源** | `source_label` | 是 | 机构、教材名、文件名 |
| **适用考试版本** | `exam_version` | 推荐 | 如 `ECO_2021`、`ECO_2025` |
| **来源路径** | `source_path` | 是 | `source_materials/...` |
| **加工版本** | `processor_version` | 否 | 本 workflow 文档版本 |
| **内容哈希** | `content_hash` | 否 | 去重与增量更新 |
| **取代关系** | `supersedes_id` | 否 | 新版资料替代旧版条目 |

### 6.1 版本规则

| 规则 | 说明 |
|------|------|
| 同 `concept_id` 重复 | **合并**或**版本追加**，不重复创建矛盾条目 |
| 新旧冲突 | 保留两者并标注 `exam_version`；默认 Agent 用较新考纲 |
| 来源不明 | `exam_version: unknown`，建议 Need Review |
| 索引 | `imported_materials.md` 记录每次导入的版本元数据 |

---

## 7. Quality Rules

### 7.1 不要（禁止）

| 禁止项 | 说明 |
|--------|------|
| **直接复制 PDF** | 不得把 PDF 全文原样粘贴进 `knowledge/` |
| **编造** | 无原文依据的题目、答案、定义、考纲权重 |
| **冒充官方** | 培训资料不得标为 Official Source |
| **静默入库** | Need Review 项不得写入正式知识条 |
| **破坏原件** | 不得修改 `source_materials/` |

### 7.2 必须（硬性）

| 必须项 | 说明 |
|--------|------|
| **结构化** | 每条知识符合 §4 或 §5 字段契约 |
| **双语核心术语** | English Term + 中文 Knowledge Name |
| **事实/推测分离** | 【事实】/【推测】/ Need Review |
| **可追溯** | `source_path` + 页码/段落 |
| **加工报告** | 每次任务输出 Report + MATERIAL_HANDOFF |

### 7.3 避免（去重与精简）

| 避免项 | 做法 |
|--------|------|
| **重复知识** | `concept_id` / `question_id` 去重；已有条目用合并策略（§6） |
| **同义堆砌** | 用 `synonym_mapping.md` 归一，不重复建条 |
| **冗长摘录** | 只保留判断所需定义与场景，非整页复制 |

### 7.4 保留（核心价值）

| 保留项 | 说明 |
|--------|------|
| **考试判断逻辑** | 决策顺序、First/Next/Best、优先级原则、陷阱模式 |
| **易混对比** | Common Trap、confusing_terms |
| **可迁移规则** | Memory Method、口诀、标准处理链 |

### 7.5 质量检查清单

- [ ] 未完成结构化 → 不入库
- [ ] OCR 低置信且无人工确认 → 仅报告，不入库
- [ ] 题库无答案 → `correct_answer: null`，不猜测为【事实】
- [ ] 已对齐 §6 版本字段
- [ ] 已更新 `imported_materials` 索引
- [ ] 原始文件未改动

---

## 8. 加工报告模板

```markdown
# Material Processing Report

| 字段 | 值 |
|------|-----|
| processing_id | |
| source_path | |
| source_class | A / B / C / D |
| content_type | |
| pmbok_version | |
| exam_version | |
| items_extracted | |
| items_stored | |
| duplicates_merged | |
| need_review_count | |

## 写入路径
- knowledge/...
- examples/...

## Need Review 清单
（无则写「无」）
```

---

## 9. 与其他模块衔接

| 模块 | 衔接 |
|------|------|
| `source_materials/README.md` | 原始资料入口 |
| `inputs/document_input.md` | 文档类路由 |
| `workflows/question_analysis.md` | 题库/错题讲解与入库 |
| `architecture/data_flow.md` | 知识数据流 |
| `knowledge/terminology/import_log.md` | 术语导入示例 |

---

## 10. Agent 执行顺序（速查）

```
1. 登记 Raw Material + source_path
2. Extract Text → Clean Content
3. Classify Topic（§2 A/B/C/D）
4. Map Knowledge Point（去重 + 对齐 glossary）
5. Generate（§4 或 §5 格式）
6. Quality Rules（§7）校验
7. Store + Version（§6）+ Report
```
