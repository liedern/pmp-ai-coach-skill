# Review Coach — 复盘聚合规则

> **版本**：`mvp-1.3`  
> **输入**：`memory/data/mistake_memory.json`、`memory/data/question_history.json`、`weak_points.json`、`learning_state.json`  
> **输出**：供 `REVIEW_RETROSPECTIVE_OUTPUT` 各字段使用

---

## 0. 数据来源（P0）

| 存储 | 在复盘中的角色 |
|------|----------------|
| **Mistake Memory** | **唯一错因分析源**：`error_type`、`error_reason`、`knowledge_point`、错因排名、高频错误类型、待复习错题清单（错因侧） |
| **Question History** | **不**提供错因分类；用于 **答题趋势**（`result` 时间序列）、**争议题**（`answer_disputed`）、**重复作答**（同 `question_id` 多条记录：先错后对、重复错等） |
| **Bookmark Memory** | **不参与**错误统计、错因排名、错误率、WeakPoint 错因聚合；**不**作为复盘必读文件 |
| **answer_disputed（History）** | v0.1.1：题库 vs Coach 分歧；**不计入** §3 错因 `share`；Mistake 不应包含「用户=Coach、仅与平台不一致」的伪错题 |
| **weak_points.json** | Mistake 已聚合的薄弱点视图（优先展示 §3） |
| **learning_state.json** | 会话与 `error_patterns` 校验、趋势辅助 |

```
错因分析 ──► 仅 mistake_memory.mistakes[]
     │
     ├─► question_history（争议标记、重复作答、正确率趋势）
     └─► ✗ bookmark_memory（错误统计禁用）
```

---

## 1. 记录分类（Mistake Memory + History 争议）

> **P0**：§2–§3 **错因与错误权重** 仅来自 `mistake_memory.mistakes[]`。  
> **History**：仅用于 §1 争议分类、§1.1 重复作答与 §7 时间窗内的 **答题趋势**（对错次数、正确率），**不得**用 History 替代 Mistake 做 `error_type` 统计。  
> **禁止**：读取 `bookmark_memory.json` 参与错因、错误率或 `domain_score`。

对 `mistake_memory.mistakes[]` 每条记录；**争议**关联 `question_history` 同 `question_id` **最新一条**的 `answer_disputed`：

| 类型 | 判定条件 | 用于统计 |
|------|----------|----------|
| **confirmed_wrong** | `user_answer` ≠ `correct_answer`，且对应 History 无 `answer_disputed` | 错因、错误权重 |
| **disputed** | History 最新条 `answer_disputed === true` | 单独列表，不计入错因排名 |
| **excluded** | `review_status === mastered` / `ignored` | 默认不入待复习 |

权重 `w`：

| 类型 | 每条记录 w |
|------|------------|
| confirmed_wrong | `2.0 × repeated_count`（无 `repeated_count` 时按 `1`） |
| disputed | `0.5`（仅知识域曝光，不进错因） |

### 1.1 Question History — 重复作答与趋势（非错因源）

对 `question_history.history[]` 按 `question_id` 分组、按 `answer_time` 排序：

| 分析项 | 规则 | 输出用途 |
|--------|------|----------|
| 重复作答 | 同 `question_id` 记录数 ≥ 2 | 「第 N 次做本题」、是否 **先错后对** |
| 答题趋势 | 时间窗内 `result=correct/wrong` 计数 | 正确率变化、复习是否见效（对比 Mistake `review_status`） |
| 争议 | 最新条 `answer_disputed` | 见上表 `disputed`，**不计入** §3 错因 `share` |

**禁止**：仅凭 History 中 `result=wrong` 新增错因条目；错因仍以 Mistake 为准。

---

## 2. 高频错误知识领域排序（§1）

### 2.1 展开标签

对每条记录的 `knowledge_point`（数组）逐标签展开；无数组时用 `["未分类"]`。

### 2.2 聚合

```
domain_score[tag] += w
domain_mistake_count[tag] += 1  （仅 confirmed_wrong）
```

> 不再累计 `domain_bookmark_count`（收藏已迁出 Mistake Memory）。

### 2.3 排序

1. 主键：`domain_score` 降序  
2. 次键：`domain_mistake_count` 降序  
3. 再次：`label` 字典序

### 2.4 输出项结构

```json
{
  "rank": 1,
  "knowledge_domain": "实施整体变更控制",
  "score": 3.5,
  "wrong_count": 1,
  "dominant_error_type": "process_sequence_error",
  "related_mistake_ids": ["mistake_..."]
}
```

`dominant_error_type`：该 tag 下 confirmed_wrong 记录的 `error_type` 众数（读时回退 `mistake_type`）；无则 `null`。

### 2.5 与 `learning_state.error_patterns.by_knowledge_point` 的关系

若 JSON 中已有 `by_knowledge_point`，复盘输出**优先以本规则重算**为准，并可附注「与 learning_state 一致 / 已重算」。

---

## 3. 高频错误类型分析（§2）

### 3.1 数据源

仅 **confirmed_wrong** 且 `error_type` 非空（读取时兼容别名 `mistake_type`）。

合并 `learning_state.error_patterns.by_error_type` 计数（若存在且时间窗为全量，取 `max(重算, 文件中)` 需一致；**默认只重算 mistake_memory，文件作校验**）。

### 3.2 MVP 错因枚举（展示用）

| 存储值 | 中文标签 |
|--------|----------|
| `knowledge_gap` | 知识盲区 |
| `concept_confusion` | 概念混淆 |
| `scenario_judgment_error` | 场景判断错误 |
| `process_order_error` | 流程顺序错误 |
| `process_sequence_error` | 流程顺序错误（Schema） |
| `keyword_misread` | 关键词误读 |
| `terminology_problem` | 术语理解问题（Schema） |
| `careless` | 粗心 |
| `carelessness` | 粗心（Schema） |

### 3.3 输出项

```json
{
  "error_type": "process_sequence_error",
  "label_zh": "流程顺序错误",
  "count": 2,
  "share": 0.67,
  "last_at": "2026-07-30T20:12:00+08:00",
  "example_knowledge_points": ["相关方管理", "需求文件"],
  "coaching_hint": "First 题先对照需求/基准，再记问题日志或升级"
}
```

`share` = 该类型 count / 全部错因 count；无错因时输出 `insufficient_data: true`。

### 3.4 `top_patterns`

从 `learning_state.error_patterns.top_patterns` 读取；若为空且某 `error_type` count ≥ 2，生成：

```json
{
  "pattern": "流程顺序类错误重复出现",
  "error_type": "process_sequence_error",
  "occurrence_count": 2
}
```

---

## 4. 当前薄弱点（§3）

### 4.1 优先来源

1. `weak_points.json` → `weak_points[]`（Mistake Coach 已聚合）  
2. 若为空：取 §2 知识域排序 **前 3 名** 且 `score >= 1.0`，合成薄弱点视图

### 4.2 合成薄弱点字段

| 字段 | 规则 |
|------|------|
| `knowledge_domain` | 标签名 |
| `priority_score` | `min(100, round(domain_score * 20))` |
| `error_trend` | 无历史 → `new`；有 `last_error_at` 可比对 sessions → `rising/stable` |
| `suggested_direction` | 见 §5 模板 |

### 4.3 待复习错题

`review_status` ∈ `new`, `learning`, `reviewing` 的记录，按 `repeated_count` 降序，附在 §3 末尾「待复习题目」子列表（`mistake_id` + 一句 `memory_rule`）。

---

## 5. 学习建议（§4）

按规则生成 3–5 条，**必须**可执行：

| 触发条件 | 建议模板 |
|----------|----------|
| `dominant_error_type` = 流程类 | 复习 `knowledge/decision_framework/pmp_priority_rules.md` — Analysis Before Action；专项 First/Next 题 5 道 |
| 知识域含「变更」 | 复习整体变更控制 + `workflows/question_analysis.md` 变更场景 |
| 知识域含「冲突/相关方」 | 复习 Collaborate Before Escalate + People 情境题 |
| `confirmed_wrong` ≥ 1 且无近期复习 | 按 `mistake_ids` 重做 3 道 |
| `disputed` ≥ 1 | 争议题单独核对题库版本，统一记「动作」不记字母 |
| 数据极少 | 建议继续做题；答错将自动进入错题库后再复盘 |

> **收藏**：考前巩固由 **Study Planner** 读 `bookmark_memory.json`；**Review 复盘不读 Bookmark、不把收藏计入错误统计**。

每条建议结构：

```json
{
  "priority": "high",
  "action": "专项练习：First 题 5 道，先团队+需求文件再日志",
  "ref": "knowledge/decision_framework/decision_tree.md"
}
```

---

## 6. 推荐下一步训练方向（§5）

输出 1 个主方向 + 0–2 个备选：

```
主方向 = weak_points[0] 或 knowledge_domain_ranking[0]
训练类型 ∈ review_mistake | weak_point_drill | knowledge_learning | mixed
```

| `training_type` | 含义 | 典型动作 |
|-----------------|------|----------|
| `review_mistake` | 错题重做 | 指定 `mistake_ids` 最多 3 条（**仅** Mistake Memory） |
| `weak_point_drill` | 薄弱域刷题 | 指定 `knowledge_domain` + 题量 |
| `knowledge_learning` | 概念巩固 | 指定 `knowledge/` 路径 |
| `mixed` | 以上组合 | 先 1 篇阅读 + 3 道题 |

```json
{
  "primary": {
    "training_type": "review_mistake",
    "title": "重做 3 道待复习错题",
    "mistake_ids": ["mistake_q856_change_control_001"],
    "estimated_minutes": 25
  },
  "alternatives": []
}
```

---

## 7. 时间窗（可选）

`time_window_days = N` 时：

- 仅统计 `created_at` / `last_wrong_at` / `updated_at` 在 `[today-N, today]` 的记录  
- `sessions` 仅取 `study_date` 在窗内的事件辅助 trend

---

## 8. 完整性检查

复盘输出前校验：

| 检查 | 失败处理 |
|------|----------|
| 各 `count` 之和 ≤ `mistakes.length` 展开合理范围 | 重算 |
| `share` 之和 ≈ 1（有错因时） | 归一化 |
| 无数据 | `data_quality: "insufficient"` |
