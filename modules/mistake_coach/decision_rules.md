# Mistake Coach — 入库决策规则

> **版本**：`mvp-1.2`（三分流 · 答错自动入库）  
> **输入**：Question Coach `QUESTION_OUTPUT`  
> **输出**：`save_decision` + `mistake_record`（对齐 `database/mistake_schema.md`）

---

## 0. P0：真实错题判定（最高优先级）

在任意入库决策之前执行：

```
normalize_answer(x):
  - 单选：大写 trim
  - 多选：排序后 JSON 数组字符串

is_real_mistake =
  user_answer != null
  AND correct_answer != null
  AND normalize_answer(user_answer) != normalize_answer(correct_answer)
```

| `is_real_mistake` | 结果 |
|-------------------|------|
| **false** | `should_save = false`，**不生成** Mistake；答对/无答案走 History 或讲解 |
| **true** | **自动** `should_save = true`（无需用户说「保存错题」） |

> **禁止**：因收藏、做对不确信、`force_save` 写入 Mistake。  
> **收藏** → Bookmark Memory（见 `question_schema.md` §4），与本模块无关。

---

## 1. 入库决策矩阵

| 条件 | `should_save` | 说明 |
|------|:-------------:|------|
| `is_real_mistake` + `error_type` + `error_reason` 齐全 | **true** | 自动入库 |
| `is_real_mistake` 但缺错因 | **true**（可先入库） | 须尽快由 Question Coach 补全；标记 `confidence_level=low` |
| `user_override.force_skip = true` | false | 用户显式跳过 |
| 答对 / 无答案 / 仅收藏 | false | 不写 Mistake；History / Bookmark 分流 |

真实错题入库时初始 `review_status`：`new`。

---

## 2. 去重与更新

读取 `memory/data/mistake_memory.json`：

| 场景 | 动作 |
|------|------|
| 无匹配 | 新建 `mistake_id`，`repeated_count = 1` |
| 再次答错 | `repeated_count += 1`，刷新错因与 `updated_at` |
| 本次答对 | 不删 Mistake；Review 可改 `review_status` |

同时：每次做题由 Question Coach / 本链路写入 **Question History**（独立文件）。

---

## 3. 字段映射（写入 Mistake 仅 canonical）

| QUESTION_OUTPUT（读） | Mistake Record（写） |
|-----------------------|----------------------|
| `user_answer` / `correct_answer` | 同名 |
| `knowledge_points` | `knowledge_point`（同值） |
| `exam_domain` 或 **只读** `eco_domain` | `exam_domain` |
| `error_type` 或 **只读** `mistake_type` | `error_type` |
| `error_reason` 或 **只读** `mistake_reason` | `error_reason` |
| — | `review_status=new`，`user_id` |

**禁止**向 `mistake_memory.json` 新写入 `mistake_type`、`mistake_reason`、`eco_domain`。

### 3.1 `error_type` 产品枚举

`knowledge_gap` | `concept_confusion` | `careless_error` | `question_reading_error` | `trap_option_error`

旧六项入库时按 `mistake_schema.md` §3.4 映射为产品五项。

---

## 4. Memory 写入

| 文件 | 条件 |
|------|------|
| `mistake_memory.json` | `should_save=true` |
| `weak_points.json` | 入库后按 Mistake **仅**聚合 |
| `question_history.json` | 有作答时（可由 Question Coach 先写） |
| `bookmark_memory.json` | **本模块不写** |

---

## 5. 用户可见确认（由 Question Coach 输出，本模块保证落盘）

答错入库成功后，用户侧已含：

- 本题已自动加入错题库  
- 错误类型 / 原因 / 知识点 / 复习建议  

---

## 6. 决策示例

```json
{ "should_save": true, "reason": "自动入库：user_answer=A ≠ correct_answer=D", "is_real_mistake": true }
```

```json
{ "should_save": false, "reason": "答对，不写 Mistake；History/Bookmark 另分流", "is_real_mistake": false }
```
