# Review Coach — 复盘聚合规则

> **版本**：`mvp-1`  
> **输入**：`memory/data/mistake_memory.json`、`weak_points.json`、`learning_state.json`  
> **输出**：供 `REVIEW_RETROSPECTIVE_OUTPUT` 各字段使用

---

## 1. 记录分类

对 `mistake_memory.mistakes[]` 每条记录划分：

| 类型 | 判定条件 | 用于统计 |
|------|----------|----------|
| **confirmed_wrong** | `is_correct === false` 或 `ingestion_tag === "wrong"` 且 `answer_disputed !== true` | 错因、错误权重 |
| **disputed** | `answer_disputed === true` | 单独列表，不计入错因排名 |
| **bookmarked** | `ingestion_tag === "bookmarked"` 且非 confirmed_wrong | 待巩固知识域 |
| **needs_review** | `ingestion_tag === "needs_review"` 且非 disputed | 待巩固 + 低权重错误信号 |

权重 `w`（用于知识域排序）：

| 类型 | 每条记录 w | 重复加成 |
|------|------------|----------|
| confirmed_wrong | `2.0 × repeated_count` | 是 |
| needs_review（非 disputed） | `1.0 × repeated_count` | 是 |
| disputed | `0.5`（仅知识域曝光，不进错因） | 否 |
| bookmarked | `0.5`（待巩固） | 否 |

---

## 2. 高频错误知识领域排序（§1）

### 2.1 展开标签

对每条记录的 `knowledge_point`（数组）逐标签展开；无数组时用 `["未分类"]`。

### 2.2 聚合

```
domain_score[tag] += w
domain_mistake_count[tag] += 1  （仅 confirmed_wrong + needs_review）
domain_bookmark_count[tag] += 1 （仅 bookmarked）
```

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
  "bookmark_count": 2,
  "dominant_mistake_type": "process_sequence_error",
  "related_mistake_ids": ["mistake_..."]
}
```

`dominant_mistake_type`：该 tag 下 confirmed_wrong 记录的 `mistake_type` 众数；无则 `null`。

### 2.5 与 `learning_state.error_patterns.by_knowledge_point` 的关系

若 JSON 中已有 `by_knowledge_point`，复盘输出**优先以本规则重算**为准，并可附注「与 learning_state 一致 / 已重算」。

---

## 3. 高频错误类型分析（§2）

### 3.1 数据源

仅 **confirmed_wrong** 且 `mistake_type` / `wrong_type` 非空。

合并 `learning_state.error_patterns.by_mistake_type` 计数（若存在且时间窗为全量，取 `max(重算, 文件中)` 需一致；**默认只重算 mistake_memory，文件作校验**）。

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
  "mistake_type": "process_sequence_error",
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

从 `learning_state.error_patterns.top_patterns` 读取；若为空且某 `mistake_type` count ≥ 2，生成：

```json
{
  "pattern": "流程顺序类错误重复出现",
  "mistake_type": "process_sequence_error",
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
| `dominant_mistake_type` = 流程类 | 复习 `knowledge/decision_framework/pmp_priority_rules.md` — Analysis Before Action；专项 First/Next 题 5 道 |
| 知识域含「变更」 | 复习整体变更控制 + `workflows/question_analysis.md` 变更场景 |
| 知识域含「冲突/相关方」 | 复习 Collaborate Before Escalate + People 情境题 |
| `bookmarked` ≥ 2 且无错因 | 对收藏题做「闭卷自测」再对照 `memory_rule` |
| `disputed` ≥ 1 | 争议题单独核对题库版本，统一记「动作」不记字母 |
| 数据极少 | 建议继续积累 10+ 道带作答的题再复盘 |

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
| `review_mistake` | 错题/收藏重做 | 指定 `mistake_ids` 最多 3 条 |
| `weak_point_drill` | 薄弱域刷题 | 指定 `knowledge_domain` + 题量 |
| `knowledge_learning` | 概念巩固 | 指定 `knowledge/` 路径 |
| `mixed` | 以上组合 | 先 1 篇阅读 + 3 道题 |

```json
{
  "primary": {
    "training_type": "review_mistake",
    "title": "重做 3 道待复习题（含变更控制收藏）",
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
