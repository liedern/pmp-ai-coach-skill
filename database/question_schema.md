# Question Schema

> 共享题目实体数据模型。`Mistake` 通过 `question_id` 关联本题。  
> **消费模块**：Question Coach（写）；Mistake Coach / Review Coach（读）。

---

## 1. Purpose

`Question` 表示题库中的一道标准题，与用户的错题记录 `Mistake` 分离：

```
Question (1) ──< (N) Mistake
```

| 用途 | 说明 |
|------|------|
| 题目归一化 | 同一题多次做错 → 同一 `question_id` |
| 数据分层 | 题目正文存 `Question`；作答、错因、复习状态存 `Mistake` |
| Web 扩展 | 多用户共享同一 `Question` 记录 |

### MVP 写入流程

Question Coach 分析完成后：

1. 创建或匹配 `Question`（按题干 + 选项）
2. 若用户确认保存 → 创建 `Mistake`，**必须**填写 `question_id`

---

## 2. MVP 字段

| 字段名称 | 类型建议 | 是否必须 | 用途 |
|----------|----------|----------|------|
| `question_id` | UUID | **是** | 主键 |
| `source` | VARCHAR(255) | 否 | 题目来源（培训机构、真题、自编等） |
| `question_text` | TEXT | **是** | 题干全文 |
| `options` | JSON | **是** | `{"A":"...","B":"...","C":"...","D":"..."}` |
| `correct_answer` | VARCHAR(32) | 否 | 正确答案；未知为 `null` |
| `knowledge_point` | VARCHAR(255) | 否 | 主知识点标签，如「冲突管理」 |
| `created_time` | TIMESTAMPTZ | **是** | 入库时间，ISO 8601 |

---

## 3. 与 Mistake 的关联

| 规则 | 说明 |
|------|------|
| 外键 | `Mistake.question_id` → `Question.question_id` |
| MVP 必填 | 保存错题时 `Mistake.question_id` **不得为 null** |
| 题目正文 | 以 `Question` 为准；`Mistake` 可保留精简快照用于离线展示 |
| 去重 | 同一题干 + 选项 → 复用已有 `question_id`，不重复创建 `Question` |

---

## 4. 与 DATA_HANDOFF 映射

| DATA_HANDOFF | Question 字段 |
|--------------|---------------|
| `question_text` | `question_text` |
| `options` | `options` |
| `correct_answer` | `correct_answer` |
| `source` | `source` |
| `knowledge_points[0]` | `knowledge_point`（取主知识点） |

PMP 分析维度（`project_type`、`eco_domain` 等）存于 `Mistake` 或 `DATA_HANDOFF`，不进入 MVP `Question` 表。

---

## 5. 最小 JSON 示例

```json
{
  "question_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "source": "模拟卷第 2 套 Q15",
  "question_text": "During project execution, two team members disagree on task assignments...",
  "options": {
    "A": "Escalate to the sponsor immediately",
    "B": "Meet with both parties to collaborate on a solution",
    "C": "Reassign tasks to avoid further conflict",
    "D": "Update the resource management plan"
  },
  "correct_answer": "B",
  "knowledge_point": "冲突管理",
  "created_time": "2026-07-28T11:00:00+08:00"
}
```

---

## 6. Future Extension（非 MVP）

以下字段在 Web 产品阶段按需扩展，P0 不实现：

`content_hash`、`exam_set`、`project_type`、`eco_domain`、`explanation`、`image_refs`、`updated_at`、`status`
