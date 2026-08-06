# Mistake Coach — `mistake_memory.json` 写入契约

> **版本**：`mvp-1.3`  
> **文件**：`memory/data/mistake_memory.json` → `mistakes[]`  
> **消费者**：Review Coach（`aggregation_rules.md` 读 `error_type`，兼容 `mistake_type`）

---

## 1. 写入条件

仅当 `decision_rules.md` **§0–§1** `is_real_mistake = true` 且通过争议门禁时追加或更新 `mistakes[]`。

对错判定、争议标记、收藏意图 **不得** 写入本文件（→ `question_history.json` / `bookmark_memory.json`）。

---

## 2. 单条记录（新写入字段）

| 字段 | 类型 | 说明 |
|------|------|------|
| `mistake_id` | string | 主键 |
| `question_id` | string | 题目指纹 |
| `user_answer` | string | 用户答案 |
| `correct_answer` | string | 当次 **adjudication_answer**（判题基准） |
| `error_type` | string | §`decision_rules.md` 产品五项 |
| `error_reason` | string | 错因说明 |
| `knowledge_point` | string[] | 知识点标签 |
| `review_status` | string | 默认 `new` |

可选扩展：`exam_domain`、`repeated_count`、`last_wrong_at`、`confidence_level`、`adjudication_source`（`coach` | `platform` | `legacy`）（见 `mistake_schema.md`）。

### 2.1 禁止新写入（历史兼容只读）

| 禁止写入 | 读取回退 |
|----------|----------|
| `mistake_type` | → `error_type` |
| `mistake_reason` | → `error_reason` |
| `eco_domain` | → `exam_domain` |
| `ingestion_tag` / `is_correct` / `answer_disputed` | 见三分流 |

**不迁移**既有 `mistakes[]` 条目中已存在的旧键名。

---

## 3. 示例（新数据）

```json
{
  "mistake_id": "mistake_stakeholder_deliverable_review_001",
  "question_id": "q_it_deliverable_stakeholder_concern_001",
  "user_answer": "D",
  "correct_answer": "B",
  "error_type": "process_order_error",
  "error_reason": "先记入问题日志，未先与团队审阅需求文件。",
  "knowledge_point": ["相关方管理", "需求文件", "Analysis Before Action"],
  "review_status": "new"
}
```

---

## 4. 禁止写入场景

| 场景 | 说明 |
|------|------|
| 答对 | 不得出现 `user_answer === correct_answer` 的新增条目 |
| 无错因 | 不得 `error_type: null` 入库（信息不足时暂不写 Mistake） |
| 仅收藏 | 写 `bookmark_memory.json` + History `result=correct` |
| 答案争议 | 写 History `answer_disputed` + `answer_evaluation`；**不**因平台 alone 污染 Mistake（`decision_rules` §0.2） |
