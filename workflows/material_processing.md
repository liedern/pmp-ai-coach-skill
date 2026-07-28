# Material Processing Workflow

> 资料加工工作流：将 `source_materials/` 中的原始 PMP 学习资料转化为 `knowledge/` 可引用的结构化知识。
>
> 本文件定义**完整加工流水线**、PMP 专项提取规则与**固定输出格式**。不替代 `skill.md` 中的教练角色定义。

---

## 1. Purpose

| 目标 | 说明 |
|------|------|
| **统一资料入口** | 多格式原始资料经同一流水线处理 |
| **结构化知识产出** | 输出标准 Markdown + `MATERIAL_HANDOFF` JSON |
| **PMP 考试适配** | 强制双语术语、考试关键词、问法信号、易混概念、场景案例 |
| **可审计** | 来源可追溯；低置信标 **Need Review**；不编造 |

**硬性规则**：`source_materials/` 中的文件**不得**被 Agent 直接引用；必须经过本工作流写入 `knowledge/` 后方可调用。

---

## 2. Supported Input Types

| 输入类型 | 扩展名 / 形式 | 预处理 | 原始存放路径（建议） |
|----------|---------------|--------|----------------------|
| **PDF** | `.pdf` | 文本层提取；扫描件 → OCR | `source_materials/{分类}/` |
| **Word** | `.docx`, `.doc` | 解析标题、表格、列表 | 同上 |
| **PPT** | `.pptx`, `.ppt` | 按幻灯片提取文本与备注；图表 → OCR | 同上 |
| **Excel** | `.xlsx`, `.xls`, `.csv` | 表头识别；行记录化 | 同上 |
| **网页** | URL / HTML / 剪藏 Markdown | 去导航/广告；保留正文与标题层级 | 用户提供 URL 或导出 HTML |
| **截图** | `.png`, `.jpg`, `.webp` | OCR → `ocr_text` + `confidence` | 同上 |

### 2.1 输入元数据

| 字段 | 说明 |
|------|------|
| `processing_id` | UUID，本次加工任务 ID |
| `input_type` | `pdf` \| `word` \| `ppt` \| `excel` \| `web` \| `image` |
| `source_path` | `source_materials/` 内相对路径或 `file_ref` |
| `source_label` | 来源说明（机构、教材、课程名） |
| `language` | `en` \| `zh` \| `bilingual` |
| `submitted_at` | ISO 8601 |

### 2.2 降级与拒绝

| 情形 | 处理 |
|------|------|
| 加密/损坏文件 | Need Review，提示重新导出 |
| OCR 低置信 | 仅保留原文片段，不推断缺失字段 |
| 网页无法抓取 | 请用户粘贴正文或导出 PDF |
| 超大文件 | 按章/按页/按 Sheet 拆分为 Processing Unit |

---

## 3. Processing Pipeline（完整流程）

**按顺序执行，不可跳步。**

```
Source Material（source_materials/）
        │
        ▼
① 内容识别（Content Recognition）
        │
        ▼
② 资料分类（Material Classification）
        │
        ▼
③ 知识提取（Knowledge Extraction）
        │
        ▼
④ 中英文关键词映射（Bilingual Keyword Mapping）
        │
        ▼
⑤ 生成结构化 Markdown（Structured Markdown Generation）
        │
        ▼
⑥ 质量校验（Quality Validation）
        │
        ▼
⑦ 写入 knowledge（Knowledge Storage）
```

---

### Step ① 内容识别（Content Recognition）

| 动作 | 说明 |
|------|------|
| 格式解析 | 按 `input_type` 提取纯文本与结构（标题、表格、列表、页码） |
| OCR（截图/扫描件） | 输出 `ocr_text`、`ocr_confidence` |
| 网页清洗 | 保留正文、标题、表格；记录 `source_url` |
| PPT 处理 | 幻灯片标题 + 要点 + 演讲者备注分别标记 |
| 切块 | 大文档 → `Processing Unit`（按章/节/卷） |
| 输出 | `raw_units[]`：每块含 `text`、`page_range`、`structure_hints` |

---

### Step ② 资料分类（Material Classification）

为每个 `Processing Unit` 判定 `primary_type`（可多标签，一主多副）：

| primary_type | 识别信号 | 写入路径 |
|--------------|----------|----------|
| `exam_material` | ECO、考试大纲、题型说明、领域权重 | `knowledge/exam/` |
| `pmbok_knowledge` | PMBOK 过程、知识领域、ITTO | `knowledge/pmbok/`、`knowledge/agile/` |
| `course_notes` | 口诀、决策套路、讲师总结 | `knowledge/decision_framework/` |
| `terminology` | 英中词汇表、缩写、对照表 | `knowledge/language/` |
| `question_bank` | 题干 + 选项、套卷、解析 | `examples/questions/` |

**分类规则**：

```
题干+选项 ≥2 → question_bank
ECO / Exam Outline → exam_material
glossary / 英中对照 → terminology
「第一步」「口诀」「讲师」→ course_notes
PMBOK 过程名 / 章节标题 → pmbok_knowledge
混合文档 → 拆分为多个 Unit，分别分类
```

---

### Step ③ 知识提取（Knowledge Extraction）

按 `primary_type` 提取字段（详见 §5）。所有字段标注：

| 标记 | 含义 |
|------|------|
| **【事实】** | 原文明确存在 |
| **【推测】** | Agent 推断，须标 `confidence` |
| **Need Review** | 无法确认，值置 `null` |

---

### Step ④ 中英文关键词映射（Bilingual Keyword Mapping）

对每条提取项**强制**执行映射（PMP 专项，见 §4）：

1. 识别英文术语 → 查 `knowledge/language/synonym_mapping.md` 归一化
2. 补全中文名称与考试含义
3. 挂载题干 **Keywords**
4. 挂载 **First / Next / Best** 问法信号词（若原文或上下文涉及）
5. 挂载 **易混概念** `confusing_terms`
6. 生成或摘录 **场景案例** `scenario_example`

无法映射的标准英文 → Need Review，不写入术语主表。

---

### Step ⑤ 生成结构化 Markdown（Structured Markdown Generation）

按 §6 **固定输出格式** 生成：

- 人类可读 Markdown 正文
- `MATERIAL_HANDOFF` JSON 块（机器可读）

---

### Step ⑥ 质量校验（Quality Validation）

| 检查项 | 规则 |
|--------|------|
| 防编造 | 无原文依据的字段不得为【事实】 |
| 双语完整 | 核心术语含 English + 中文 + exam_meaning |
| 路径正确 | `target_path` 与 `primary_type` 一致 |
| 去重 | 同 `concept_id` / `question_id` 合并或版本追加 |
| Need Review | 未解决项不写入 `knowledge/` 主内容 |

---

### Step ⑦ 写入 knowledge（Knowledge Storage）

| 动作 | 说明 |
|------|------|
| 写入 | 追加或合并至目标 `.md` 文件 |
| 索引 | 在 `knowledge/source/imported_materials.md` 登记 |
| 保留原件 | **不删除** `source_materials/` 原文件 |
| 报告 | 输出 §6.3 加工报告 |

---

## 4. PMP 专项加工要求

加工每条知识时，**必须尝试填充**以下五类 PMP 增强字段（原文无则标 Need Review 或【推测】，禁止编造）：

### 4.1 中英文术语解析（Bilingual Term Parsing）

每条核心概念：

```markdown
**English Term（中文名称）**
- 中文解释：
- PMP考试含义：
- 英文原词保留：是
```

对齐输出：`knowledge/language/pmp_terms.md` 条目结构。

### 4.2 高频考试关键词（Exam Keywords）

从原文或上下文提取题干信号词，写入 `keywords[]`：

| 类别 | 示例 |
|------|------|
| 过程词 | change request, risk register, baseline, CCB |
| 场景词 | conflict, resistance, scope creep, variance |
| 角色词 | Product Owner, Scrum Master, sponsor |

参考：`knowledge/language/keyword_mapping.md`、`knowledge/exam/exam_keywords_mapping.md`。

### 4.3 First / Next / Best 题型关键词（Question Intent Keywords）

若资料涉及答题方法、讲师口诀或例题解析，提取问法信号：

| 问法 | 英文信号 | 考试含义 | 优先动作倾向 |
|------|----------|----------|--------------|
| First | first, initially | 找第一步 | analyze, document, meet |
| Next | next, then, following | 流程下一步 | 状态机 +1 |
| Best | best, most effective | 多选择优 | priority_rules |
| Most Appropriate | most appropriate | 情境最贴合 | 排除过度/不足 |
| Should | should, recommended | PMI 推荐行为 | 流程 + 协作 |

写入字段：`question_intent_keywords[]`。

### 4.4 易混概念（Confusing Terms）

每条知识至少检查是否与已知易混项关联：

```yaml
confusing_terms:
  - term: Issue Log
    distinction: 已发生问题；Risk Register 为未来风险
```

参考：`knowledge/language/confusing_terms.md`。

### 4.5 场景案例（Scenario Examples）

摘录或改写原文中的情境/例题（不改写为考题答案，除非原文含解析）：

```yaml
scenario_example:
  context: 两名团队成员因工作分配发生冲突
  signal_words: [conflict, team, work allocation]
  exam_focus: First 题 — 合作解决优先于升级
  source_page: 42
```

---

## 5. Extraction Rules by Type

### 5.1 知识点（pmbok / exam / course）

| 字段 | 键名 | 必须 |
|------|------|------|
| 概念 | `concept` | 是 |
| 定义 | `definition` | 是 |
| 过程 | `process` | 否 |
| 知识领域 | `knowledge_domain` | 推荐 |
| 关键词 | `keywords[]` | 推荐 |
| 问法关键词 | `question_intent_keywords[]` | 推荐 |
| 易混概念 | `confusing_terms[]` | 推荐 |
| 场景案例 | `scenario_example` | 推荐 |
| 考试陷阱 | `exam_trap` | 推荐 |
| 来源页码 | `source_page` | 否 |

### 5.2 术语（terminology）

| 字段 | 键名 | 必须 |
|------|------|------|
| 英文术语 | `english_term` | 是 |
| 中文名称 | `chinese_term` | 是 |
| 考试含义 | `exam_meaning` | 是 |
| 关键词 | `keywords[]` | 推荐 |
| 场景 | `scenario` | 推荐 |
| 易混概念 | `confusing_terms[]` | 推荐 |
| 示例 | `example` | 推荐 |

### 5.3 题目（question_bank）

| 字段 | 键名 | 必须 |
|------|------|------|
| 题干 | `question` | 是 |
| 选项 | `options` | 是 |
| 正确答案 | `correct_answer` | 原文有则必填；无则 null |
| 知识点 | `knowledge_points[]` | 推荐 |
| 推理 | `reasoning` | 推荐 |
| 问法 | `question_intent` | 推荐 |
| 陷阱模式 | `trap_patterns[]` | 推荐 |

### 5.4 课程口诀（course_notes）

| 字段 | 键名 | 必须 |
|------|------|------|
| 规则原话 | `rule_statement` | 是 |
| 优先级顺序 | `priority_order[]` | 推荐 |
| 适用类型 | `applies_to` | 推荐 |
| 反例 | `counterexample` | 否 |

---

## 6. Output Format（明确输出格式）

### 6.1 单条知识 — Markdown 模板

```markdown
---
id: {concept_id}
type: {primary_type}
source: {source_label}
source_path: source_materials/{path}
source_page: {page}
confidence: high | medium | low
target_path: knowledge/{...}
imported_at: {ISO8601}
---

## {English Term}（{中文名称}）

### Definition
【事实|推测】{definition}

### Process
{process}

### PMP Exam Context
{exam_meaning / 考试怎么考}

### Keywords
- {keyword_1}
- {keyword_2}

### Question Intent Keywords
| 问法 | 信号词 | 判断要点 |
|------|--------|----------|
| First | … | … |

### Confusing Terms
| 易混概念 | 区分要点 |
|----------|----------|
| {term} | {distinction} |

### Scenario Example
> 【事实】{scenario_example}

### Exam Trap
{exam_trap}

### Source
- 原始文件：`source_materials/...`
- 页码：{page}
```

### 6.2 单条术语 — 表格行（批量词汇表）

```markdown
| English Term | 中文名称 | PMP考试含义 | Keywords | 易混概念 | Example |
|--------------|----------|-------------|----------|----------|---------|
| Validate Scope | 确认范围 | 客户正式验收可交付成果 | Customer Acceptance, Deliverable | Control Quality | … |
```

### 6.3 单条题目 — Markdown 模板

```markdown
---
id: {question_id}
type: question_bank
source: {source_label}
exam_set: {set_name}
confidence: high | medium | low
---

## Question
{question_text}

## Options
- A. …
- B. …
- C. …
- D. …

## Correct Answer
【事实】{answer} | Need Review

## Knowledge Points
- …

## Question Intent
First | Next | Best | …

## Reasoning
【事实|推测】…

## Trap Patterns
- T02 过早升级
```

### 6.4 MATERIAL_HANDOFF JSON（机器交接块）

每条加工任务末尾**必须**附加：

````markdown
<!-- MATERIAL_HANDOFF:BEGIN -->
```json
{
  "processing_id": "uuid",
  "source_path": "source_materials/textbook/PMBOK_ch7.pdf",
  "source_label": "PMBOK 第7章",
  "input_type": "pdf",
  "primary_type": "pmbok_knowledge",
  "units_processed": 3,
  "items": [
    {
      "item_id": "uuid",
      "item_type": "knowledge_point",
      "concept": "Validate Scope（确认范围）",
      "english_term": "Validate Scope",
      "chinese_term": "确认范围",
      "definition": "【事实】…",
      "process": "Validate Scope",
      "knowledge_domain": "scope",
      "keywords": ["Customer Acceptance", "Deliverable"],
      "question_intent_keywords": [
        { "intent": "first", "signals": ["first", "initially"], "hint": "验收前确认交付" }
      ],
      "confusing_terms": [
        { "term": "Control Quality", "distinction": "查质量是否符合标准，非正式验收范围" }
      ],
      "scenario_example": {
        "context": "…",
        "signal_words": [],
        "exam_focus": "…"
      },
      "exam_trap": "勿与 Control Quality 混淆",
      "confidence": "high",
      "target_path": "knowledge/pmbok/scope.md",
      "need_review": false
    }
  ],
  "stored_paths": ["knowledge/pmbok/scope.md"],
  "need_review_items": [],
  "processed_at": "2026-07-28T14:00:00+08:00"
}
```
<!-- MATERIAL_HANDOFF:END -->
````

### 6.5 加工报告（固定输出）

```markdown
# Material Processing Report

| 字段 | 值 |
|------|-----|
| processing_id | |
| source_path | source_materials/... |
| input_type | pdf / word / ppt / excel / web / image |
| primary_type | |
| units_processed | |
| items_extracted | |
| items_stored | |
| need_review_count | |
| stored_paths | [] |
| warnings | [] |

## Need Review 清单
（无则写「无」）

## 写入摘要
- `knowledge/...`：+N 条
```

---

## 7. Human Review

### 7.1 触发条件

- OCR/解析置信度低
- 术语英文不确定
- 题库无正确答案
- 分类冲突
- 与已有 `knowledge/` 内容矛盾

### 7.2 规则

```json
{
  "review_id": "uuid",
  "field": "correct_answer",
  "reason": "原文未提供",
  "raw_snippet": "…",
  "status": "pending"
}
```

- **不得**用猜测填充为【事实】
- Need Review 项**不写入** `knowledge/` 正文，仅出现在报告中

---

## 8. Storage Mapping（source → knowledge）

| source_materials/ | primary_type | knowledge/ 目标 |
|-------------------|--------------|-----------------|
| `exam/` | exam_material | `knowledge/exam/` |
| `textbook/` | pmbok_knowledge | `knowledge/pmbok/`、`knowledge/agile/` |
| `course/` | course_notes | `knowledge/decision_framework/` |
| `terminology/` | terminology | `knowledge/language/` |
| `question_bank/` | question_bank | `examples/questions/` |

索引登记：`knowledge/source/imported_materials.md`。

---

## 9. Agent 检查清单

- [ ] 输入来自 `source_materials/` 或用户上传（登记 `source_path`）
- [ ] 完成 ①–⑦ 全流程
- [ ] 每条知识含 PMP 五类增强（§4）：术语、考试关键词、问法词、易混概念、场景案例
- [ ] 输出符合 §6 Markdown + MATERIAL_HANDOFF 格式
- [ ] Need Review 项未静默入库
- [ ] 原始文件未删除
- [ ] 已输出加工报告

---

## 10. 流水线总览

```
source_materials/
        │
        ▼
① 内容识别 ── PDF/Word/PPT/Excel/网页/截图
        │
        ▼
② 资料分类 ── exam | pmbok | course | terminology | question_bank
        │
        ▼
③ 知识提取 ── concept | term | question | rule
        │
        ▼
④ 中英文关键词映射 ── language/ 体系对齐
        │
        ▼
⑤ 结构化 Markdown ── §6 模板
        │
        ▼
⑥ 质量校验 ── 防编造 | 双语 | 去重
        │
        ▼
⑦ 写入 knowledge/ + imported_materials 索引
```
