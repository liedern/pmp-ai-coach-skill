# Mistake Coach（错题教练模块）

> **所属产品**：PMP AI Coach（单一主 Skill）  
> **模块类型**：专业能力模块 | **非独立 Skill**

---

## 模块目标

负责 **真实错题**（用户答错）的沉淀、错因归类、重复错误识别与薄弱点上升，将单题分析结果转化为 Review Coach 可聚合的学习记忆。

- 接收 Question Coach 的 `DATA_HANDOFF`；**仅当** `user_answer` ≠ `correct_answer` 时写入 `mistake_memory`
- 维护 `mistake_memory`（含 `error_type`、`error_reason`、`knowledge_point`、`review_status`、`repeated_count`）
- 聚合生成 `weak_points` 薄弱域报告
- 为 Study Planner / Review Coach 提供优先级输入

**不负责**：单题逐步讲解（→ Question Coach）、日计划排期（→ Study Planner）。

---

## 用户触发场景

| 场景 | 典型用户说法 |
|------|--------------|
| 加入错题本 | 「加入错题本」——**答错已自动入库**；用于强调复习，非二次保存 |
| 做错后自动归档 | Question Coach 输出且满足 P0 真实错题 |
| 查询薄弱点 | 「我哪方面最弱？」「错误模式是什么？」 |
| 查看错题统计 | 「冲突管理我错几次了？」 |
| 更新复习状态 | 「这道题我掌握了」「标记为已复习」 |
| 重复做错 | 同一 `question_id` 再次答错 |

---

## 输入

| 输入类型 | 来源 | 说明 |
|----------|------|------|
| 结构化错题 | Question Coach `DATA_HANDOFF` | 首选入库路径 |
| 用户错题导出 | `source_materials/`（D 类资料） | 经 `material_processing` 加工 |
| 历史记忆 | `memory/mistake_memory.md` | 去重、累加 `repeated_count` |
| 用户档案 | `memory/user_profile.md` | 掌握内容过滤 |

**必填字段（真实错题入库）**：`question_id`、`user_answer`、`correct_answer`、`error_type`、`error_reason`、`knowledge_point`、`review_status`；**禁止**新写 `mistake_type` / `eco_domain`（读时兼容）。

---

## 输出

| 产出 | 格式 | 说明 |
|------|------|------|
| 错题记忆条目 | `mistake_memory` 单条 JSON/YAML | 对齐 `database/mistake_schema.md` |
| 薄弱点聚合 | `weak_points` 记录列表 | `error_count`、`error_trend`、`suggested_direction` |
| 模式摘要 | 自然语言报告 | 高频 `error_type`、重复知识点 |
| 复习优先级 | `priority_score` | 供 Review Coach 排序 |

### MVP 实现（`mvp-1.1`）

| 文件 | 说明 |
|------|------|
| `module.md` | P0 门槛、Memory 写入、MISTAKE_OUTPUT 契约 |
| `decision_rules.md` | P0、入库矩阵、去重、`error_type` 枚举与映射 |
| `memory_record_contract.md` | `mistake_memory.json` 单条结构 |
| `examples/sample_mistake_record.json` | 对齐 `mistake_schema.md` 的记录示例 |
| `examples/sample_handoff.json` | 完整 MISTAKE_OUTPUT 示例 |

---

## 数据依赖

### 工作流（Workflows）

| 文件 | 关系 |
|------|------|
| `workflows/mistake_classification.md` | 入库与 Memory 更新（含 P0） |
| `workflows/question_analysis.md` §5–§6 | 错因枚举与保存决策 |

### 知识库（Knowledge）

| 路径 | 用途 |
|------|------|
| `knowledge/language/confusing_terms.md` | Concept Confusion 关联 |
| `knowledge/exam/trap_patterns.md` | 错因与陷阱映射 |

### 数据模型（Database）

| 文件 | 关系 |
|------|------|
| `database/mistake_schema.md` | **主 Schema**（字段、状态机、错因枚举） |

### 记忆（Memory）

| 文件 | 读写 |
|------|------|
| `memory/mistake_memory.md` | **读/写** |
| `memory/data/mistake_memory.json` | **读/写**（MVP 运行时） |
| `memory/data/weak_points.json` | **写**（聚合） |
| `memory/data/learning_state.json` | **写**（学习状态） |
| `memory/weak_points.md` | **写**（聚合） |
| `memory/user_profile.md` | 读 |
| `memory/learning_progress.md` | 写（错题相关进度） |

### 上游模块

| 模块 | 关系 |
|------|------|
| `modules/question_coach/` | 接收 `DATA_HANDOFF` |

### 下游模块

| 模块 | 关系 |
|------|------|
| `modules/review_coach/` | 提供待复习错题列表 |
| `modules/study_planner/` | 提供薄弱域与优先级 |

---

## 模块边界

```
Question Coach (DATA_HANDOFF)
        │
        ▼
Mistake Coach ──→ mistake_memory
        │
        ├──聚合──→ weak_points
        │
        └──输出──→ Review Coach / Study Planner
```
