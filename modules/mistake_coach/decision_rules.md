# Mistake Coach — 入库决策规则

> **版本**：`mvp-1.3`（三分流 · 答错自动入库 · Answer Validation v0.1.1）  
> **输入**：Question Coach `QUESTION_OUTPUT`（含 `answer_evaluation`）  
> **输出**：`save_decision` + `mistake_record`（对齐 `database/mistake_schema.md`）

---

## 0. Answer Validation（AEL · v0.1.1）

在 **P0 真实错题判定** 之前，从 handoff 读取 `answer_evaluation`（缺失则按 **legacy**，见 §0.4）。

### 0.1 `adjudication_answer`（判题唯一基准）

```
若 coach_answer 非空 且 answer_confidence ∈ { high, medium }:
    adjudication_answer = coach_answer
否则若 platform_answer 非空:
    adjudication_answer = platform_answer
否则:
    adjudication_answer = null
```

- **`platform_answer`**：题库 / App / 用户粘贴解析中的标答（【事实·来源】，非绝对真理）。  
- **`coach_answer`**：按 PMI 考试逻辑独立推断的应选项；须配 `coach_answer_reason`。  
- **`answer_confidence`**：仅针对 `coach_answer`：`high` / `medium` / `low`。  
- **`answer_status`**（最小枚举）：`coach_only` | `aligned` | `disputed` | `uncertain` | `legacy`。  
  - `platform_answer` 与 `coach_answer` 均非空且归一化后不等 → `disputed`，**`answer_disputed=true`**。  
  - 无 `platform_answer` → `coach_only`。  
  - 相等 → `aligned`。  
  - `coach_answer` 为空或仅 `low` 置信且无可靠 Coach → `uncertain`。

Handoff 中 **`correct_answer`** 与 **`adjudication_answer` 同义**（v0.1.1 写入时二者一致）。

### 0.2 争议不污染 Mistake Memory

| 条件 | `should_save` |
|------|:-------------:|
| `answer_disputed=true` 且 `user_answer` 归一化等于 `coach_answer`，且 `answer_confidence` ∈ { high, medium } | **false**（用户按考试逻辑选对；不因题库标答记错） |
| `answer_status=uncertain` 或 `adjudication_answer=null` | **false** |
| `user_override.force_skip=true` | **false** |
| 其余见 §1 | 按 `is_real_mistake` |

争议记录只体现在 **Question History**（`answer_disputed`、`answer_evaluation`、`correct_answer_note`）；**禁止**向 `mistake_memory.json` 写入 `answer_disputed`。

### 0.3 用户纠偏话术（更新 History / 重算 adjudication）

| 用户说 | 动作 |
|--------|------|
| 「以你的判断为准」「按考试逻辑我对了」 | `adjudication_answer=coach_answer`；按 §1 重算是否 Mistake |
| 「以题库为准」 | `adjudication_answer=platform_answer`；`correct_answer_note` 注明用户采纳平台；按 §1 重算 |

### 0.4 Legacy（v0.1.0 无 `answer_evaluation`）

- `adjudication_answer` ← handoff `correct_answer`  
- `answer_status=legacy`，`answer_disputed` 仅读 History 顶层（若有）

---

## 1. P0：真实错题判定

在 §0 之后执行：

```
normalize_answer(x):
  - 单选：大写 trim
  - 多选：排序后 JSON 数组字符串

is_real_mistake =
  user_answer != null
  AND adjudication_answer != null
  AND normalize_answer(user_answer) != normalize_answer(adjudication_answer)
```

再应用 **§0.2** 争议门禁。

| `is_real_mistake` 且通过 §0.2 | 结果 |
|-------------------------------|------|
| **false** | `should_save = false` |
| **true** | **自动** `should_save = true`（无需用户说「保存错题」） |

> **禁止**：因收藏、`force_save` 写入 Mistake。  
> **收藏** → Bookmark Memory（见 `question_schema.md` §4）。

---

## 2. 入库决策矩阵

| 条件 | `should_save` | 说明 |
|------|:-------------:|------|
| `is_real_mistake` + §0.2 通过 + `error_type` + `error_reason` 齐全 | **true** | 自动入库 |
| `is_real_mistake` 但缺错因 | **true**（可先入库） | 标记 `confidence_level=low` |
| `user_override.force_skip = true` | false | 用户显式跳过 |
| 答对 / 无 adjudication / 仅收藏 | false | History / Bookmark 分流 |

写入 Mistake 时 **`correct_answer` = `adjudication_answer`**；`answer_confidence=low` 时 Mistake 可选 `confidence_level=low`。  
可选 **`adjudication_source`**：`coach` | `platform` | `legacy`。

真实错题入库时初始 `review_status`：`new`。

---

## 3. 去重与更新

读取 `memory/data/mistake_memory.json`：

| 场景 | 动作 |
|------|------|
| 无匹配 | 新建 `mistake_id`，`repeated_count = 1` |
| 再次答错 | `repeated_count += 1`，刷新错因与 `updated_at` |
| 本次答对 | 不删 Mistake；Review 可改 `review_status` |

同时：每次做题由 Question Coach / 本链路写入 **Question History**（独立文件）。

---

## 4. 字段映射（写入 Mistake 仅 canonical）

| QUESTION_OUTPUT（读） | Mistake Record（写） |
|-----------------------|----------------------|
| `user_answer` | 同名 |
| `adjudication_answer` 或 `correct_answer` | `correct_answer` |
| `knowledge_points` | `knowledge_point`（同值） |
| `exam_domain` 或 **只读** `eco_domain` | `exam_domain` |
| `error_type` 或 **只读** `mistake_type` | `error_type` |
| `error_reason` 或 **只读** `mistake_reason` | `error_reason` |
| `answer_evaluation.answer_confidence=low` | `confidence_level=low`（可选） |
| — | `review_status=new`，`user_id` |

**禁止**向 `mistake_memory.json` 新写入 `mistake_type`、`mistake_reason`、`eco_domain`、`answer_disputed`。

### 4.1 `error_type` 产品枚举

`knowledge_gap` | `concept_confusion` | `careless_error` | `question_reading_error` | `trap_option_error`

旧六项入库时按 `mistake_schema.md` §3.4 映射为产品五项。

---

## 5. Memory 写入

| 文件 | 条件 |
|------|------|
| `mistake_memory.json` | `should_save=true` |
| `weak_points.json` | 入库后按 Mistake **仅**聚合 |
| `question_history.json` | 有作答时（含 `answer_evaluation` 快照） |
| `bookmark_memory.json` | **本模块不写** |

---

## 6. 用户可见确认（由 Question Coach 输出，本模块保证落盘）

答错入库成功后，用户侧已含：

- 本题已自动加入错题库  
- 错误类型 / 原因 / 知识点 / 复习建议  

争议且未入库时：说明「本题题库标答与考试逻辑判断不一致，已按 Coach 判断记录，未记入错题库」。

---

## 7. 决策示例

```json
{ "should_save": true, "reason": "自动入库：user_answer=A ≠ adjudication_answer=D (coach)", "is_real_mistake": true }
```

```json
{ "should_save": false, "reason": "争议：用户答案与 coach_answer 一致，不因 platform 记错", "is_real_mistake": false }
```

```json
{ "should_save": false, "reason": "答对（对 adjudication），不写 Mistake", "is_real_mistake": false }
```
