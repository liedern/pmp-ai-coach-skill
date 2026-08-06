# Mistake Classification Workflow

> 错题分类与入库工作流：接收 Question Coach 输出，决策是否入库，生成 Mistake 记录，更新 Memory。
>
> **单题错因分类**在 Question Coach（`workflows/question_analysis.md` §5）完成；本工作流负责**入库决策、记录生成、模式聚合**。

---

## 0. 执行原则

| 原则 | 说明 |
|------|------|
| 上游优先 | 以 `QUESTION_OUTPUT` 为事实来源，不重新推理答案 |
| P0 真实错题 | `user_answer` ≠ **`adjudication_answer`** 才入库（见 `decision_rules.md` §0–§1） |
| 不编造 | 缺 `error_type` / `error_reason` 不入库 |
| 幂等去重 | 同题更新，不重复创建 |
| MVP 无库 | 写入 `memory/data/*.json`，不调用数据库 |

---

## 1. Trigger（触发条件）

| 触发场景 | 识别信号 |
|----------|----------|
| Question Coach 完成分析 | 收到 `DATA_HANDOFF` → 先执行 **§2.5 P0**，再决策 |
| 用户说「加入错题本」 | 走 `question_analysis`；**答错已自动入库**，答对则不入库 |
| 用户查询薄弱点 | 「我哪方面最弱」→ 读取 Memory，不走入库 |
| 用户更新复习状态 | 「这道题掌握了」→ 更新 `mistake_memory`，本工作流 §6 |

---

## 2. Input Validation

校验 `QUESTION_OUTPUT`：

| 检查项 | 失败处理 |
|--------|----------|
| `module === "question_coach"` | 拒绝处理，提示上游模块错误 |
| `question_recognition.question_text` 非空 | 标记 `insufficient_information`，建议用户补充 |
| `question_recognition.options` 至少 2 项 | 同上 |
| `version` 兼容 | 当前支持 `mvp-1` |

---

## 2.5 Real Mistake Gate（P0 + AEL）

按 `modules/mistake_coach/decision_rules.md` **§0 Answer Validation** 与 **§1 P0**：

```
读取 answer_evaluation → adjudication_answer
应用 §0.2 争议门禁
normalize(user_answer) != normalize(adjudication_answer)
  AND adjudication_answer non-null
        │
        ├─ false → should_save=false，输出原因，结束
        └─ true  → 继续 §3
```

---

## 3. Save Decision

按 `modules/mistake_coach/decision_rules.md` **§1** 执行（在 P0 通过后）：

```
读取 user_override + error_type / error_reason 完整性
        │
        ▼
应用决策矩阵 → should_save
        │
        ├─ false → 输出 MISTAKE_OUTPUT（mistake_record=null），结束
        │
        └─ true → 继续 §4
```

---

## 4. Dedup & Record Build

### Step 4.1 去重检测

```
读取 memory/data/mistake_memory.json
        │
        ▼
匹配 question_id 或题干指纹
        │
        ├─ 无匹配 → action=create
        └─ 有匹配 → action=update, repeated_count+=1
```

### Step 4.2 构建 Mistake 记录

按 `decision_rules.md` §3 映射字段，生成 `mistake_record`：

- 新增：分配 `mistake_id`、`question_id`
- 更新：保留原 `mistake_id`，刷新 `error_type`、`error_reason`、`updated_at`、`last_wrong_at`（**禁止**新写 `mistake_type` / `mistake_reason` / `eco_domain`）

完整结构见 `database/mistake_schema.md` 与 `modules/mistake_coach/examples/sample_mistake_record.json`。

---

## 5. Memory Update

### Step 5.1 写入 mistake_memory.json

```json
{
  "mistake_id": "...",
  "question_id": "...",
  "user_answer": "A",
  "correct_answer": "B",
  "error_type": "scenario_judgment_error",
  "error_reason": "...",
  "knowledge_point": ["冲突管理"],
  "review_status": "new"
}
```

结构权威定义见 `modules/mistake_coach/memory_record_contract.md`。对错与争议同时写入 `question_history.json`（`result` / `answer_disputed`）。

### Step 5.2 重算 weak_points.json

```
按 knowledge_point 分组
error_count = sum(repeated_count) 或 记录条数
dominant_error_type = 组内 error_type 众数（读时回退 mistake_type）
error_trend = 与上次聚合比较（new / rising / stable / falling）
priority_score = min(100, error_count * 15 + repeated_bonus)
```

### Step 5.3 追加 learning_state.json

在 `sessions[]` 追加：

```json
{
  "event": "mistake_saved",
  "mistake_id": "...",
  "knowledge_points": ["冲突管理"],
  "error_type": "scenario_judgment_error",
  "error_type": "scenario_judgment_error",
  "timestamp": "2026-07-29T18:05:00+08:00"
}
```

更新 `error_patterns` 计数（按 `error_type` 与 `knowledge_point`）。

---

## 6. Review Status Update（用户触发）

用户说「掌握了」「忽略」时：

| 用户指令 | 更新 |
|----------|------|
| 「掌握了」 | `review_status → mastered`（需最近一次重做正确，或用户确认） |
| 「忽略」 | `review_status → ignored` |
| 「再复习」 | `review_status → reviewing`，`review_count += 1` |

---

## 7. Pattern Insight（模式识别）

入库后检查：

```
IF repeated_count >= 2
   OR 同 knowledge_point 的 error_count >= 3
   OR 同 error_type 近 7 天 >= 2
THEN 生成 pattern_insight
```

输出自然语言摘要，供 Agent 在回复中引用（不暴露模块名）。

---

## 8. Fixed Output

每次 Mistake Coach 处理完成后输出：

```markdown
## 错题归档

- **是否入库**：是 / 否
- **原因**：[save_decision.reason]
- **错题 ID**：[mistake_id / 无]
- **重复错误**：是（第 N 次）/ 否

[若有 pattern_insight，附一句个性化提示]
```

末尾附 `MISTAKE_HANDOFF` JSON 块（见 `modules/mistake_coach/module.md` §4）。

---

## 9. 工作流总览

```
QUESTION_OUTPUT
        │
        ▼
  ┌─────────────┐
  │ 0. P0 Gate  │ ← user_answer ≠ correct_answer
  └──────┬──────┘
         ▼
  ┌─────────────┐
  │ 1. Validate │
  └──────┬──────┘
         ▼
  ┌─────────────┐
  │ 2. Decision │ ← decision_rules.md
  └──────┬──────┘
         │
    ┌────┴────┐
    ▼         ▼
  skip      save
    │         │
    │         ▼
    │   ┌──────────┐
    │   │ 3. Dedup │
    │   └────┬─────┘
    │        ▼
    │   ┌──────────────┐
    │   │ 4. Build     │
    │   │    Mistake   │
    │   └──────┬───────┘
    │          ▼
    │   ┌──────────────┐
    │   │ 5. Memory    │
    │   │    Update    │
    │   └──────┬───────┘
    │          ▼
    │   ┌──────────────┐
    │   │ 6. Pattern   │
    │   └──────┬───────┘
    │          │
    └────┬─────┘
         ▼
  MISTAKE_OUTPUT
```

---

## 10. 与其他模块的关系

| 模块 | 关系 |
|------|------|
| Question Coach | 上游，提供 `QUESTION_OUTPUT` |
| Review Coach | 下游，消费 `mistake_memory` 执行复习 |
| Study Planner | 下游，读取 `weak_points` 排任务 |
| Memory | 本工作流直接读写 `memory/data/` |
