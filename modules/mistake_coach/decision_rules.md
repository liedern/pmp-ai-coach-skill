# Mistake Coach — 入库决策规则

> **版本**：`mvp-1`  
> **输入**：Question Coach `QUESTION_OUTPUT`  
> **输出**：`save_decision` + `mistake_record`（对齐 `database/mistake_schema.md`）

---

## 1. 入库决策矩阵

### 1.1 主决策表

| 条件 | `should_save` | `ingestion_tag` | 初始 `review_status` |
|------|---------------|-----------------|----------------------|
| `review_status = explain_only` 且用户未强制保存 | **false** | — | — |
| `user_override.force_skip = true` | **false** | — | — |
| `user_override.force_save = true` | **true** | `bookmarked` | `new` |
| `review_status = wrong` | **true** | `wrong` | `new` |
| `review_status = needs_review` | **true** | `needs_review` | `new` 或 `learning` |
| `review_status = bookmarked` | **true** | `bookmarked` | `new` |

### 1.2 建议确认后再入库

以下情况 `should_save` 可为 `true`，但 Agent **应询问用户**：

| 条件 | 说明 |
|------|------|
| `correct_answer_confidence` 为 `low` | 正确答案不确定 |
| `pending_confirmations` 非空 | 题干/选项有缺失 |
| `question_recognition.confidence` 为 `low` | OCR 质量差 |

用户确认后入库；用户拒绝则 `should_save = false`。

### 1.3 不入库但需说明

`should_save = false` 时，`save_decision.reason` 必须写明原因，例如：

- 「用户选择仅讲解，未要求保存」
- 「无用户答案，无法形成有效错题记录」

---

## 2. 去重与更新规则

读取 `memory/data/mistake_memory.json`，按以下优先级匹配：

```
1. question_id 完全匹配
2. question_text 归一化后相似度 ≥ 0.95（Agent 判断）
```

| 场景 | 动作 |
|------|------|
| 无匹配 | 创建新 `mistake_id`，`repeated_count = 1` |
| 有匹配且再次答错 | 更新记录：`repeated_count += 1`，刷新 `wrong_type`、`updated_at`；`review_status` 从 `mastered` 回退至 `reviewing` |
| 有匹配且答对 | 不新增；可选更新 `review_status` 向 `mastered` 推进（Review Coach 职责，MVP 仅记录事件） |

### 2.1 `question_id` 生成（MVP）

无主题库时，使用内容指纹：

```
question_id = "q_" + hash(normalize(question_text) + sorted(options))
```

Agent 可用题干前 40 字 + 选项 A 摘要作为可读 ID，如 `q_conflict_team_001`。

---

## 3. 字段映射

### 3.1 Question Coach → Mistake Schema

| QUESTION_OUTPUT | `mistake_schema` 字段 |
|-----------------|----------------------|
| `question_recognition.question_text` | `question_text` |
| `question_recognition.options` | `options` |
| `question_recognition.user_answer` | `user_answer` |
| `correct_answer` | `correct_answer` |
| `question_recognition.source` | `source` |
| `question_recognition.exam_set` | `exam_set` |
| `question_recognition.project_approach` | `project_type` |
| `question_recognition.project_phase` | `project_phase` |
| `eco_domain` | `eco_domain` |
| `knowledge_points` | `knowledge_points` |
| `process` | `process` |
| `explanation` | `answer_explanation` |
| `correct_answer_confidence` | `confidence_level` |
| `mistake_reason` | `wrong_reason` |
| `explanation.memory_rule` | → `mistake_memory.memory_rule` |

### 3.2 MVP 错因类型映射

| Question Coach `mistake_type` | Mistake `wrong_type` |
|-------------------------------|----------------------|
| `knowledge_gap` | `knowledge_gap` |
| `concept_confusion` | `concept_confusion` |
| `scenario_judgment_error` | `scenario_judgment_error` |
| `process_order_error` | `process_sequence_error` |
| `keyword_misread` | `terminology_problem` |
| `careless` | `carelessness` |

### 3.3 `review_status` 映射

工作流 `review_status`（入库意图）→ Mistake 表 `review_status`（复习生命周期）：

| 工作流 `review_status` | Mistake `review_status` |
|------------------------|-------------------------|
| `wrong` | `new` |
| `needs_review` | `new` |
| `bookmarked` | `new` |

`ingestion_tag` 保留原始入库原因（写入 `mistake_record.metadata.ingestion_tag` 或 `mistake_memory` 扩展字段）。

---

## 4. 薄弱点聚合触发

每次 `should_save = true` 且写入 `mistake_memory.json` 后：

1. 读取全部错题
2. 按 `knowledge_points` 主标签分组
3. 重算 `weak_points.json`（规则见 `memory/weak_points.md` §6）
4. 更新 `learning_state.json` 中的 `error_patterns` 与 `last_session`

---

## 5. 决策输出示例

### 5.1 答错 — 入库

```json
{
  "should_save": true,
  "reason": "用户答错（A≠B），review_status=wrong",
  "ingestion_tag": "wrong"
}
```

### 5.2 仅讲解 — 不入库

```json
{
  "should_save": false,
  "reason": "review_status=explain_only，用户未要求保存",
  "ingestion_tag": null
}
```

### 5.3 重复错题 — 更新

```json
{
  "should_save": true,
  "reason": "匹配已有 question_id=q_conflict_001，更新重复错误记录",
  "ingestion_tag": "wrong",
  "action": "update",
  "previous_repeated_count": 1
}
```
