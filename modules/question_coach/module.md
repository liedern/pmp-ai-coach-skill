# Question Coach — MVP 模块实现

> **所属产品**：PMP AI Coach（单一主 Skill）  
> **模块类型**：专业能力模块 | **非独立 Skill**  
> **执行工作流**：`workflows/question_analysis.md`（步骤与推理）  
> **本文件职责**：MVP 输入/输出契约、七项结构化分析、与下游交接

---

## 1. 模块定位

Question Coach 是「有题可判」场景的分析引擎，完成：

```
输入（截图 / 文字 / 可选用户答案）
        │
        ▼
题目识别 + PMP 推理 + 结构化讲解
        │
        ▼
DATA_HANDOFF JSON ──→ Mistake Coach
```

**不负责**：错题入库决策（→ Mistake Coach）、薄弱点聚合（→ Mistake Coach + Memory）。

---

## 2. 输入契约

### 2.1 支持输入

| 输入类型 | 来源 | 预处理 |
|----------|------|--------|
| 题目截图 | `inputs/image_input.md` | OCR → 结构化题干与选项 |
| 题目文字 | `inputs/question_input.md` | 直接结构化 |
| 用户答案（可选） | 用户消息 | 无答案时 `user_answer = null` |

### 2.2 输入 JSON（`QUESTION_INPUT`）

Agent 在模块入口将原始输入归一化为：

```json
{
  "input_type": "question",
  "input_channel": "text | image",
  "question_text": "题干全文",
  "options": { "A": "...", "B": "...", "C": "...", "D": "..." },
  "user_answer": null,
  "correct_answer": null,
  "official_explanation": null,
  "source": null,
  "exam_set": null,
  "attempt_date": null,
  "user_intent": "analyze | save | explain_only",
  "ocr_metadata": null
}
```

| 字段 | 必须 | 说明 |
|------|------|------|
| `question_text` | 是 | 题干；OCR 不完整时保留已识别部分 |
| `options` | 是 | 至少一个选项；缺失项列入 `pending_confirmations` |
| `user_answer` | 否 | 未提供则为 `null`，**不得**推断用户答案 |
| `user_intent` | 否 | 默认 `analyze`；用户说「不用保存」→ `explain_only` |

### 2.3 读取上下文（可选）

| 文件 | 用途 |
|------|------|
| `memory/data/mistake_memory.json` | 检测重复错题、引用历史错因 |
| `memory/data/learning_state.json` | 个性化讲解深度 |
| `knowledge/decision_framework/*` | PMP 推理 |
| `knowledge/language/*` | 术语解释 |

---

## 3. 执行流程

```
1. Input Normalize     ← QUESTION_INPUT
2. PMP Framework       ← workflows/question_analysis.md §3
3. Answer Explanation  ← §4
4. Terminology & Plain ← 本模块 §4（术语 + 大白话）
5. Mistake Classify    ← 有 user_answer 且答错时
6. Fixed Output        ← workflows/question_analysis.md §7
7. Data Handoff        ← QUESTION_OUTPUT（本模块 §5）
```

推理与讲解步骤**必须**完整执行 `workflows/question_analysis.md`，本模块仅补充 MVP 要求的**术语解释**与**大白话理解**字段。

---

## 4. MVP 七项结构化分析

每次分析必须在用户可见输出与 `QUESTION_OUTPUT` 中覆盖以下七项：

| # | 输出项 | 用户可见位置 | JSON 字段 |
|---|--------|--------------|-----------|
| 1 | **题目识别** | §1 题目信息 | `question_recognition` |
| 2 | **正确答案** | §2 正确答案 | `correct_answer` + `correct_answer_confidence` |
| 3 | **每个选项分析** | §6 选项逐项分析 | `explanation.option_analysis` |
| 4 | **PMP 知识点** | §4 考察知识点 | `knowledge_points` + `exam_domain` + `process` |
| 5 | **专业术语解释** | 新增 §「专业术语」 | `terminology_explanations` |
| 6 | **大白话理解** | 新增 §「大白话理解」 | `plain_language_summary` |
| 7 | **错因判断**（答错时） | §7 用户错因 | `error_type` + `error_reason` |

### 4.1 题目识别（`question_recognition`）

```json
{
  "question_text": "【事实】题干原文",
  "options": { "A": "...", "B": "...", "C": "...", "D": "..." },
  "question_intent": "first | next | best | most_appropriate | should_do | implicit",
  "project_approach": "predictive | agile | hybrid | unknown",
  "project_phase": "initiating | planning | executing | monitoring_controlling | closing | unknown",
  "confidence": "high | medium | low",
  "pending_confirmations": ["source", "exam_set"]
}
```

### 4.2 专业术语解释（`terminology_explanations`）

从题干与选项中提取 1–5 个影响判断的 PMP 术语，每项：

```json
{
  "term": "Collaborate",
  "term_zh": "协作/合作",
  "definition": "与相关方共同协商以达成共识",
  "exam_signal": "冲突、分歧、团队问题时优先考虑面对面沟通",
  "ref": "knowledge/language/pmp_terms.md"
}
```

优先引用 `knowledge/language/pmp_terms.md`、`confusing_terms.md`；无匹配时基于 PMP 框架解释并标【推测】。

### 4.3 大白话理解（`plain_language_summary`）

用非术语、场景化语言总结「这题在考什么、该怎么想」，80–150 字，面向零基础考生。

示例：

> 两个人吵架了，项目经理不是第一时间找老板，而是先让俩人坐下来谈清楚。PMP 认为：能自己解决就先自己解决，谈不拢再往上报。

### 4.4 错因分类（产品五项 · `error_type`）

用户答错时，`error_type` **必须**从以下枚举选一；说明写入 `error_reason`：

| 存储值 | 中文 | 典型情形 |
|--------|------|----------|
| `knowledge_gap` | 知识缺失 | 不了解该过程/工具/实践 |
| `concept_confusion` | 概念混淆 | 两个相近概念选错 |
| `careless_error` | 粗心 | 手滑、漏看限定词 |
| `question_reading_error` | 题干理解错误 | 看错 First/Best/NOT 或英文术语 |
| `trap_option_error` | 选项陷阱 | 落入干扰项套路 / 场景判断失误 |

| 条件 | `error_type` / `error_reason` |
|------|-------------------------------|
| 无 `user_answer` | `null` / `null` |
| 答对 | `null` / `null` |
| 答错 | `error_type` 必选 + `error_reason` 必填 |

答错后：**History 记录 → 自动写入 Mistake**；**禁止**询问「是否保存错题」。

---

## 5. 输出契约（`QUESTION_OUTPUT` / DATA_HANDOFF）

分析完成后，在 Markdown 末尾输出 `DATA_HANDOFF` 块，结构对齐本契约。

完整字段定义见 `output_contract.md`；最小示例如 `examples/sample_output.json`。

### 5.1 核心字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `module` | string | 固定 `"question_coach"` |
| `version` | string | 契约版本，当前 `"mvp-1"` |
| `question_recognition` | object | §4.1 |
| `correct_answer` | string \| null | 正确答案选项 |
| `correct_answer_confidence` | string | `high` / `medium` / `low` |
| `explanation` | object | 含 `option_analysis`、`memory_rule` 等 |
| `knowledge_points` | string[] | PMP 知识点标签 |
| `terminology_explanations` | array | §4.2 |
| `plain_language_summary` | string | §4.3 |
| `error_type` | string \| null | §4.4 产品五项 |
| `error_reason` | string \| null | 错因说明 |
| `data_routing` | object | `write_history` / `write_mistake` / `write_bookmark` |
| `review_status` | string | `wrong` / `correct` / `explain_only` |
| `created_at` | string | ISO 8601 |

### 5.2 下游分流（三分流）

`QUESTION_OUTPUT` 完成后按规则分流（**禁止**再询问「是否保存错题」）：

| 条件 | 动作 |
|------|------|
| 存在 `user_answer` | 写 **Question History** |
| `user_answer` ≠ `correct_answer`（均非空） | **自动**进入 Mistake Coach → Mistake Memory；用户可见：「本题已自动加入错题库」 |
| 用户主动「收藏这题」等 | 写 **Bookmark Memory** |
| 仅讲解、无作答 | History 可选跳过；不写 Mistake |

Mistake Coach 读取后：

1. P0 校验真实错题并入库（`decision_rules.md`）
2. 生成 Mistake Record（`mistake_schema.md`）
3. 更新 `memory/data/mistake_memory.json` + `weak_points.json`

---

## 6. 用户可见输出补充段落

在 `workflows/question_analysis.md` §7 固定模板中，于「## 6. 选项逐项分析」之后插入：

```markdown
## 6.5 专业术语

| 术语 | 含义 | 考试信号 |
|------|------|----------|
| ... | ... | ... |

## 6.6 大白话理解

> [plain_language_summary 正文]
```

其余章节编号顺延（原 §7 用户错因 → 保持为 §7）。

---

## 7. Agent 约束

| 约束 | 说明 |
|------|------|
| 不编造 | 无用户答案不判错因；无官方解析不冒充 |
| 事实分离 | 【事实】/【推测】/【待确认】 |
| 不直接写 Memory | History / Mistake / Bookmark 按 `data_routing` 分流写入；答错自动 Mistake，**禁止**询问保存 |
| 不暴露模块名 | 对用户始终以「PMP AI Coach」口吻输出 |

---

## 8. 文件索引

| 文件 | 说明 |
|------|------|
| `module.md` | 本文件 |
| `output_contract.md` | `QUESTION_OUTPUT` 完整 JSON Schema 说明 |
| `examples/sample_output.json` | 端到端输出示例 |
| `workflows/question_analysis.md` | 推理与讲解主流程 |
| `README.md` | 模块概览（路由与边界） |
