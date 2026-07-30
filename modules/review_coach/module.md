# Review Coach — MVP 模块实现

> **所属产品**：PMP AI Coach（单一主 Skill）  
> **模块类型**：专业能力模块 | **非独立 Skill**  
> **版本**：`mvp-1`  
> **数据层**：只读 `memory/data/*`（MVP 无数据库）

---

## 1. 模块定位

Review Coach 消费 Mistake Coach 写入的记忆数据，把「积累」转化为「可执行的复习与复盘」。

**MVP 第一能力：学习复盘（用户说「复盘」）**

```
用户：「复盘」
        │
        ▼
读取 memory/data/
  ├── mistake_memory.json
  ├── weak_points.json
  └── learning_state.json
        │
        ▼
聚合 + 分析（aggregation_rules.md）
        │
        ▼
固定五段输出 + REVIEW_RETROSPECTIVE_OUTPUT
        │
        ▼
可选写入 memory/data/review_retrospective.json（最近一次复盘快照）
```

**不负责**：单题讲解（→ Question Coach）、错题入库（→ Mistake Coach）、多周排期（→ Study Planner）。

**后续能力（本 MVP 不实现执行细节）**：今日复习清单、重做打卡、`review_status` 更新 — 见 `README.md` 与 `database/mistake_schema.md` §4。

---

## 2. 触发条件

| 用户输入 | 路由 | 工作流 |
|----------|------|--------|
| **复盘** | Review Coach | `workflows/review_retrospective.md` |
| 复盘 / 学习总结 / 我错在哪 / 错误模式 | 同上 | 同上 |
| 今天复习什么 / 打卡 / 标记掌握 | Review Coach（复习执行，P1） | 待扩展 |

**P0 硬规则**：用户仅说「复盘」时，**不**进入 Question Coach；**不**要求用户选题。

---

## 3. 输入契约

### 3.1 必读文件

| 文件 | 用途 |
|------|------|
| `memory/data/mistake_memory.json` | 错题/收藏明细、`knowledge_point`、`mistake_type`、`repeated_count` |
| `memory/data/weak_points.json` | Mistake Coach 已聚合的薄弱点（可为空） |
| `memory/data/learning_state.json` | `error_patterns`、`stats`、`sessions` |

### 3.2 可选上下文

| 文件 | 用途 |
|------|------|
| `memory/user_profile.md` | 考试日期、学习阶段（个性化建议） |
| `modules/mistake_coach/decision_rules.md` | 错因枚举与 `wrong_type` 映射 |

### 3.3 输入 JSON（`REVIEW_INPUT`，Agent 内部）

```json
{
  "trigger": "retrospective",
  "user_utterance": "复盘",
  "user_id": "default_user",
  "options": {
    "include_bookmarked": true,
    "include_disputed": true,
    "time_window_days": null
  }
}
```

| 字段 | 默认 | 说明 |
|------|------|------|
| `include_bookmarked` | `true` | 收藏题计入「待巩固知识域」，不计入错因统计 |
| `include_disputed` | `true` | 争议题单独列出，不强行计错 |
| `time_window_days` | `null` | `null` = 全量；可设 7/30 做近期复盘 |

---

## 4. 输出契约（五段 + JSON）

用户可见输出**必须**包含以下五节（顺序固定）：

| # | 章节标题 | 对应 JSON 字段 |
|---|----------|----------------|
| 1 | 高频错误知识领域排序 | `knowledge_domain_ranking` |
| 2 | 高频错误类型分析 | `mistake_type_analysis` |
| 3 | 当前薄弱点 | `weak_points_snapshot` |
| 4 | 学习建议 | `learning_suggestions` |
| 5 | 推荐下一步训练方向 | `next_training` |

完整字段见 `output_contract.md`；示例见 `examples/sample_retrospective_output.json`。

末尾附：

````markdown
<!-- REVIEW_RETROSPECTIVE:BEGIN -->
```json
{ ... }
```
<!-- REVIEW_RETROSPECTIVE:END -->
````

---

## 5. 数据不足时的行为

| 情况 | 行为 |
|------|------|
| `mistakes` 为空 | 输出空报告 + 说明「尚无错题记录，请先做题并保存」 |
| 无 `mistake_type` 非空记录 | §2 标明「暂无确认错因」；用知识域出现频次 + 收藏/待复习题补充 §1、§3 |
| `weak_points` 为空 | §3 由 `aggregation_rules.md` 从 `mistake_memory` **现场计算** |
| 仅收藏、无真错题 | §1 以「待巩固知识域」呈现；§2 说明错因样本不足 |

**禁止**：编造未出现在 memory 中的错题或统计数字。

---

## 6. 写入 Memory（可选）

复盘结束后可更新：

| 文件 | 写入内容 |
|------|----------|
| `memory/data/review_retrospective.json` | 最近一次 `REVIEW_RETROSPECTIVE_OUTPUT` 快照 |
| `memory/data/learning_state.json` | `sessions[]` 追加 `event: retrospective_completed`；`stats.total_reviews++` |

Question Coach / Mistake Coach **不**参与复盘写入逻辑。

---

## 7. Agent 约束

| 约束 | 说明 |
|------|------|
| 只读聚合 | 复盘默认不改 `mistake_memory`（除非用户另说「标记掌握」） |
| 事实来源 | 数字必须来自 JSON 或规则重算，与展示一致 |
| 用户无感 | 回复中不出现「Review Coach」模块名 |
| 知识库引用 | 建议可指向 `knowledge/` 路径，不虚构章节 |

---

## 8. 文件索引

| 文件 | 说明 |
|------|------|
| `module.md` | 本文件 |
| `aggregation_rules.md` | 排序与统计规则 |
| `output_contract.md` | `REVIEW_RETROSPECTIVE_OUTPUT` |
| `examples/sample_retrospective_output.json` | 示例（基于当前 memory 样例数据逻辑） |
| `workflows/review_retrospective.md` | 「复盘」执行步骤 |
| `README.md` | 模块概览与边界 |
