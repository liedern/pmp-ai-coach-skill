# Study Planner — 规划规则

> **版本**：`mvp-1`  
> **输入**：`mistake_memory.json`、`weak_points.json`、`learning_state.json`  
> **参考**：`modules/review_coach/aggregation_rules.md`（知识域权重）、`modules/mistake_coach/decision_rules.md` §0（真实错题）

---

## 1. 记录分类（与 Review Coach 一致）

| 类型 | 判定 | 用于计划 |
|------|------|----------|
| **confirmed_wrong** | P0 真实错题 | 最高优先级复习 |
| **pending_review** | `review_status` ∈ `new` / `reviewing` / `learning` | 纳入复习队列 |
| **disputed** | `question_history` 同 `question_id` 最新条 `answer_disputed=true` | 低权重曝光，不进错因 |
| **excluded** | `mastered` / `ignored` | 默认不排入今日 |

> Bookmark 不在 Mistake Memory。考前可从 `bookmark_memory.json` **额外**加一条 `item_type` 辅助任务，权重低于 confirmed_wrong。

---

## 2. 知识领域得分（`domain_score`）

对每条 Mistake 的 `knowledge_point`（或 `knowledge_points`）展开标签：

```
w_base:
  confirmed_wrong     → 3.0 × repeated_count
  pending_review      → 2.0 × repeated_count
  disputed            → 0.5

domain_score[tag] += w_base
```

可选辅助（不进入错因分）：

```
bookmark_boost[tag] = min(1.0 × bookmark_count_for_tag, 2.0)  // 仅用于「考前收藏」任务排序
```

若存在 `memory/data/review_retrospective.json` 且 **`snapshot_status === "current"`**，且 `source_revisions.mistake_memory_updated_at` **等于** 当前 `mistake_memory.json` 的 `updated_at`，则：

```
domain_score[tag] += retrospective_score[tag] × 0.5   // 融合复盘结果
```

否则 **跳过快照融合**，仅基于 `mistake_memory` + `weak_points` 计算（避免过期快照污染计划）。

**今日重点知识领域**：取 `domain_score` Top **3**（用户 `focus_domain` 指定时置顶）。

---

## 3. ECO 领域权重（考试大纲）

`knowledge/exam/ECO.md` 未填充时，使用 **PMP ECO 默认占比**（Process / People / Business Environment）：

| `exam_domain` | 权重 `eco_w`（读时 `exam_domain ?? eco_domain`） |
|--------------|--------------|
| `process` | 0.50 |
| `people` | 0.42 |
| `business_environment` | 0.08 |
| `unknown` | 0.33（均分） |

单条 Mistake 若有 `exam_domain`（或历史 `eco_domain`），在计算 **薄弱点优先级** 时：

```
eco_boost = 1 + eco_w   // 例如 process 条目 ×1.5
priority_component = domain_score[tag] × eco_boost
```

---

## 4. 错误类型加权（`error_type`）

仅 **confirmed_wrong** 统计 `error_type`（读时回退 `mistake_type`）。

| `error_type` | 训练侧重 | 题量系数 `type_k` |
|--------------|----------|-------------------|
| `knowledge_gap` | 概念阅读 + 基础题 | 1.2 |
| `concept_confusion` | 对比表 + 易混题 | 1.3 |
| `scenario_judgment_error` | First/Next 专项 | 1.4 |
| `process_order_error` | 流程顺序口诀 | 1.3 |
| `keyword_misread` | 关键词清单 | 1.1 |
| `careless` | 少量重做 + 检查清单 | 0.8 |

全局主导错因 `top_error_type` = 众数（无则 `null`）。

`learning_state.error_patterns.by_error_type` 存在时，与重算结果 **取较大计数** 合并。

---

## 5. 薄弱点优先级（`weak_point_priorities`）

### 5.1 数据源优先级

```
1. weak_points.json → weak_points[]（已有 priority_score 则直接用）
2. 否则由 domain_score + eco_boost 生成 synthetic 条目
3. review_retrospective.weak_points_snapshot 作校验
```

### 5.2 `priority_score`（0–100，无 weak_points 时）

```
priority_score = min(100, round(
  40 × normalize(domain_score[tag]) +
  30 × (repeated_count_sum / max_repeated) +
  20 × eco_w × 100 +
  10 × (type_k if tag tied to top_error_type else 1.0)
))
```

排序：主键 `priority_score` 降序；次键 `domain_score` 降序。

---

## 6. 距考试日期（`days_to_exam`）

读取 `memory/user_profile.json` → `exam_date`。

```
days_to_exam = exam_date - plan_date   // 日历天；无 exam_date → null

节奏系数 pace:
  days_to_exam == null     → pace = 1.0
  days_to_exam > 60        → pace = 0.85   // 夯实基础，阅读占比↑
  30 < days_to_exam ≤ 60   → pace = 1.0
  14 < days_to_exam ≤ 30   → pace = 1.15  // 刷题+复习↑
  days_to_exam ≤ 14        → pace = 1.25  // 冲刺：复习为主，可提示模考
```

`study_stage`（`learning_state.current_study_stage` 或 profile）调整：

| `study_stage` | 复习:练习:阅读 默认比例 |
|---------------|-------------------------|
| `not_started` / `foundation` | 20% : 40% : 40% |
| `practice` | 40% : 45% : 15% |
| `review` | 55% : 35% : 10% |
| `sprint` | 50% : 40% : 10% |

---

## 7. 推荐复习题数量

```
base_review = confirmed_wrong 条数（review_status 非 mastered）
bookmark_aux = min(3, bookmark_memory 条数)   // 辅助，不计入错误

raw_count = (base_review × 1.0) × pace
若存在 bookmark_aux 且 days_to_exam ≤ 14：
  raw_count += bookmark_aux × 0.3

若 raw_count < 1 且无 Mistake：
  raw_count = 5   // 通用日；可提示做题积累真实错题
  若有 Bookmark：可推荐「复习收藏重点题」N 道

recommended_review_question_count = clamp(round(raw_count × type_k_global), 3, 25)

其中 type_k_global = top_error_type 对应 type_k，无则 1.0
```

拆分输出：

- `mistake_redo_count` = ceil(recommended × 0.6)
- `drill_count` = recommended - mistake_redo_count

---

## 8. 时间分配（`estimated_study_minutes`）

```
available = options.available_minutes
       ?? user_profile.daily_study_time
       ?? 90

estimated_study_minutes = round(available × pace)   // 上限 180

块分配：
  review_block   = 35% × estimated
  drill_block    = 40% × estimated
  reading_block  = 25% × estimated

按 study_stage 比例表调整三块占比后，再映射到 focus_knowledge_domains[].allocated_minutes
```

单题复习预估 **6 分钟/道**（含自讲）；阅读材料用 `recommended_materials[].estimated_minutes`。

---

## 9. 推荐学习材料（规则）

按 **Top 知识域** 与 **top_error_type** 映射（路径须为仓库内真实文件）：

| 条件 | 推荐路径 |
|------|----------|
| 变更/整合/流程 | `knowledge/decision_framework/decision_tree.md` |
| 敏捷/价值/待办 | `knowledge/decision_framework/pmp_priority_rules.md` |
| 术语/关键词 | `knowledge/language/pmp_terms.md`、`knowledge/language/confusing_terms.md` |
| 场景判断 / First | `knowledge/decision_framework/decision_tree.md` |
| 陷阱模式 | `knowledge/exam/trap_patterns.md` |
| 默认 | `knowledge/exam/question_patterns.md` |

每条材料 `estimated_minutes`：短文档 10–15，决策树 15–20。

`mistake_review` 类型材料：`target_refs.mistake_ids` = 按优先级选的 ID 列表（不超过 `mistake_redo_count`）。

---

## 10. 生成 `study_plan_items`（同日任务）

按优先级生成 **3–6 条**：

| 顺序 | `item_type` | 条件 |
|------|-------------|------|
| 1 | `review_mistake` | `mistake_redo_count > 0` |
| 2 | `knowledge_learning` | 至少有 1 条 `recommended_materials` |
| 3 | `weak_point_drill` | `weak_points` 非空或 synthetic 薄弱域 Top1 |
| 4 | `review_mistake` 或 drill | 补足 `recommended_review_question_count` |
| 5 | `mock_exam` | `days_to_exam ≤ 14` 且 `include_mock_exam`（P1 提示项） |

`priority` 从 100 递减。

---

## 11. 数据质量标签

| 条件 | `data_quality` |
|------|----------------|
| `confirmed_wrong_count ≥ 1` 且（`weak_point_count ≥ 1` 或 domain 聚合清晰） | `good` |
| 仅有 bookmark / partial / 无错因 | `partial` |
| `mistake_count === 0` | `insufficient` |

---

## 12. 禁止行为

- 不得虚构 `mistake_id` 或错题次数  
- 不得修改其他模块 JSON（仅可选写 `daily_study_plan.json`）  
- 无 `exam_date` 时不得断言「考前 N 天」
