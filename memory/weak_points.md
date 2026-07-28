# Weak Points（薄弱点分析）

> 个人学习记忆系统 — 从错题记忆中聚合的个人薄弱领域视图。
>
> 本文档定义 `weak_points` 的数据结构。可由 Agent 定期从 `mistake_memory` 聚合生成，或由用户手动标注。

---

## 1. Purpose

将分散的错题记录**上升为可行动的薄弱点报告**，驱动：

| 用途 | 说明 |
|------|------|
| 短板可视化 | 用户清楚「弱在哪」 |
| 学习优先级 | `study_plan`、每日任务优先排弱项 |
| 趋势监控 | 错误变多/变少，调整策略 |
| 个性化建议 | 每个薄弱域给出具体学习方向 |

### 数据来源

```
mistake_memory（错题明细）
        │ 按 knowledge_point / eco_domain / mistake_type 聚合
        ▼
weak_points（薄弱点汇总）
        │
        ▼
user_profile.mastered_content（排除已掌握）
learning_progress.next_plan（写入下一步）
```

---

## 2. Core Fields

每条薄弱点记录包含以下字段：

| 字段名称 | 类型建议 | 是否必须 | 用途 |
|----------|----------|----------|------|
| `knowledge_domain` | STRING | **是** | 知识领域标识与展示名（见 §3） |
| `error_count` | INTEGER | **是** | 该领域累计错误次数（或错题条数） |
| `last_error_at` | TIMESTAMPTZ | 否 | 最近一次错误时间 |
| `error_trend` | ENUM | 推荐 | 错误趋势（见 §4） |
| `suggested_direction` | TEXT / STRING[] | 推荐 | 建议学习方向与行动项 |

### 建议附加字段

| 字段名称 | 用途 |
|----------|------|
| `weak_point_id` | 薄弱点记录 ID |
| `user_id` | 所属用户 |
| `eco_domain` | People / Process / Business Environment |
| `dominant_mistake_type` | 该领域最高频错因 |
| `related_question_ids` | 关联错题 ID 列表（抽样） |
| `priority_score` | 复习优先级分数（越高越优先） |
| `updated_at` | 聚合计算时间 |

---

## 3. knowledge_domain（知识领域）

可从以下维度打标，**同一薄弱点选主维度**：

| 维度类型 | 示例 `knowledge_domain` |
|----------|-------------------------|
| PMBOK 知识领域 | `scope`, `schedule`, `risk`, `stakeholder`, `integration` … |
| ECO 领域 | `people`, `process`, `business_environment` |
| 敏捷专题 | `scrum_roles`, `agile_change`, `servant_leadership` |
| 决策能力 | `first_vs_best`, `collaborate_before_escalate`, `change_control` |
| 术语 | `terminology_risk_issue`, `terminology_contract` |

展示时建议：`label`（中文）+ `tag`（英文存储值）。

---

## 4. error_trend（错误趋势）

| 存储值 | 中文 | 判定规则（建议） |
|--------|------|------------------|
| `rising` | 上升 | 近 7 天错误次数 > 前 7 天，或 `repeated_count` 增加 |
| `stable` | 持平 | 两周期错误次数相近 |
| `falling` | 下降 | 近 7 天错误少于前 7 天，且复习中有做对 |
| `new` | 新发现 | 首次出现在薄弱点列表 |
| `resolved` | 已改善 | 连续 N 次做对或用户/系统标记掌握 |

---

## 5. suggested_direction（建议学习方向）

每条薄弱点至少 1 条可执行建议，格式建议：

```json
{
  "suggested_direction": [
    "复习 knowledge/decision_framework/pmp_priority_rules.md § Collaborate Before Escalate",
    "专项练习：冲突管理类情境题 10 道",
    "记忆规则：团队冲突先面对面，再升级"
  ]
}
```

| 建议类型 | 示例 |
|----------|------|
| 知识复习 | 指向 `knowledge/pmbok/risk.md` 章节 |
| 决策规则 | 指向 `decision_framework` 口诀 |
| 练习任务 | 同类题 X 道、模考章节 |
| 术语巩固 | `terminology/pmp_glossary` 易混词 |

**禁止**：编造用户未做过的统计；`error_count` 必须来自 `mistake_memory` 聚合或用户确认。

---

## 6. 聚合规则（Agent 计算）

```
1. 从 mistake_memory 读取当前用户全部错题
2. 按 knowledge_point 主标签分组（一题可贡献多个薄弱点）
3. error_count = 组内记录数，或 sum(repeated_count)
4. last_error_at = max(last_wrong_at)
5. dominant_mistake_type = 组内 mistake_type 众数
6. error_trend = 按 §4 时间窗口比较
7. priority_score = f(error_count, last_error_at, exam_target_date 临近度)
8. 排除 user_profile.mastered_content 中 confidence=high 的标签
9. 生成 suggested_direction（结合 dominant_mistake_type + 知识库路径）
```

---

## 7. 单条记录示例

```json
{
  "knowledge_domain": "conflict_management",
  "label": "冲突管理",
  "eco_domain": "people",
  "error_count": 5,
  "last_error_at": "2026-07-28T09:30:00+08:00",
  "error_trend": "rising",
  "dominant_mistake_type": "scenario_judgment_error",
  "suggested_direction": [
    "重读 pmp_priority_rules：Collaborate Before Escalate",
    "完成 People 领域冲突类练习题 5 道",
    "复盘 mistake_memory 中 repeated_count ≥ 2 的题目"
  ],
  "priority_score": 85
}
```

---

## 8. 数据模板（运行时填充）

```yaml
weak_points: []
# 每条见 §7 结构
last_aggregated_at: null
```
