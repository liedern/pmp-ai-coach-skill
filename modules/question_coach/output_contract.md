# Question Coach — 输出契约（QUESTION_OUTPUT）

> **版本**：`mvp-1`  
> **消费者**：Mistake Coach、`memory/data/*`（经 Mistake Coach 写入）  
> **格式**：Markdown 末尾 `DATA_HANDOFF` JSON 块

---

## 1. 交接块格式

````markdown
<!-- DATA_HANDOFF:BEGIN -->
```json
{ ... }
```
<!-- DATA_HANDOFF:END -->
````

---

## 2. 顶层结构

```json
{
  "module": "question_coach",
  "version": "mvp-1",
  "question_recognition": { },
  "correct_answer": "B",
  "correct_answer_confidence": "high",
  "explanation": { },
  "knowledge_points": [],
  "exam_domain": "people",
  "process": "管理团队",
  "terminology_explanations": [],
  "plain_language_summary": "",
  "error_type": null,
  "error_reason": null,
  "review_status": "explain_only",
  "created_at": "2026-07-29T18:00:00+08:00",
  "pending_confirmations": []
}
```

---

## 3. 字段定义

### 3.1 `question_recognition`

| 字段 | 类型 | 必须 | 说明 |
|------|------|------|------|
| `question_text` | string | 是 | 题干全文 |
| `options` | object | 是 | `{"A":"...","B":"..."}` |
| `question_intent` | string | 是 | `first` / `next` / `best` / `most_appropriate` / `should_do` / `implicit` |
| `project_approach` | string | 是 | `predictive` / `agile` / `hybrid` / `unknown` |
| `project_phase` | string | 是 | 见 `question_analysis.md` §3 |
| `confidence` | string | 是 | `high` / `medium` / `low` |
| `source` | string \| null | 否 | 题目来源 |
| `exam_set` | string \| null | 否 | 套卷编号 |
| `user_answer` | string \| null | 否 | 用户答案 |
| `ocr_metadata` | object \| null | 否 | `{ "confidence": "high", "image_ref": "..." }` |

### 3.2 `explanation`

| 字段 | 类型 | 必须 | 说明 |
|------|------|------|------|
| `correct_answer_reason` | string | 是 | 正确答案成立原因 |
| `pmp_logic_summary` | string | 是 | 考试逻辑一句话 |
| `option_analysis` | object | 是 | 键为选项字母，值为分析结论 |
| `standard_sequence` | string[] | 否 | 标准处理顺序 |
| `memory_rule` | string | 是 | 一句话记忆规则 |
| `trap_patterns` | string[] | 否 | 干扰项陷阱类型 |

### 3.3 `terminology_explanations[]`

| 字段 | 类型 | 必须 | 说明 |
|------|------|------|------|
| `term` | string | 是 | 英文或中文术语 |
| `term_zh` | string | 否 | 中文对照 |
| `definition` | string | 是 | 简明定义 |
| `exam_signal` | string | 是 | 题干/选项中的信号 |
| `ref` | string | 否 | 知识库路径 |

### 3.4 `error_type` / `error_reason`

产品五项（答错必填）。**新 DATA_HANDOFF 只写** `error_type`、`error_reason`。

**读取兼容**（旧 handoff / 历史 JSON）：若仅有 `mistake_type` / `mistake_reason`，视为与 canonical 同义。

```
knowledge_gap
concept_confusion
careless_error
question_reading_error
trap_option_error
null   # 无用户答案或答对
```

### 3.5 `review_status` / `data_routing`

| `review_status` | 含义 |
|-----------------|------|
| `explain_only` | 仅讲解、无作答 |
| `wrong` | 答错 → **自动** Mistake |
| `correct` | 答对 → History only（除非收藏） |

| `data_routing` 字段 | 含义 |
|---------------------|------|
| `write_history` | 有作答则为 `true` |
| `write_mistake` | 答错为 `true`（自动） |
| `write_bookmark` | 用户主动收藏为 `true` |

**禁止**：用 `bookmarked` 作为 Mistake 入库标签；收藏走 Bookmark Memory。

### 3.6 用户可见归档提示（答错时必出）

```markdown
## 归档

- **本题已自动加入错题库**
- **错误类型**：{error_type}
- **错误原因**：{error_reason}
- **知识点**：{knowledge_points}
- **后续复习建议**：{一句可执行建议}
```

**禁止**输出：「需要的话回复保存错题」。

## 4. 与 `question_analysis.md` 映射

| question_analysis 字段 | QUESTION_OUTPUT 字段 |
|------------------------|----------------------|
| `question_text` | `question_recognition.question_text` |
| `options` | `question_recognition.options` |
| `project_approach` | `question_recognition.project_approach` |
| `project_phase` | `question_recognition.project_phase` |
| `exam_domain` | `exam_domain`（读兼容 `eco_domain`） |
| `error_type` | 产品五项 |
| `error_reason` | 错因说明 |
| `explanation` | `explanation` |
| `review_status` | `review_status` |
| — | `terminology_explanations`（本契约新增） |
| — | `plain_language_summary`（本契约新增） |

---

## 5. 校验规则

| 规则 | 说明 |
|------|------|
| R1 | `module` 必须为 `"question_coach"` |
| R2 | `options` 至少 2 个键 |
| R3 | `user_answer` 为 null 时，`error_type` 必须为 null |
| R4 | `error_type` 非 null 时，`error_reason` 必填 |
| R5 | `write_mistake=true` 时必须 `user_answer` ≠ `correct_answer` 且 `error_type` 非 null |
| R8 | **禁止**在新 `QUESTION_OUTPUT` 中写入 `mistake_type`、`mistake_reason`、`eco_domain` |
| R6 | 缺失值用 `null`，不用空字符串 |
| R7 | 禁止要求用户确认「保存错题」；答错自动入库 |
