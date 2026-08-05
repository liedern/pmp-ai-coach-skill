# Review Retrospective Workflow（学习复盘）

> 用户触发 **「复盘」** 时执行。  
> **读取** Mistake Coach / Memory 数据，**不**接数据库。  
> **模块契约**：`modules/review_coach/module.md`

---

## 0. 执行原则

| 原则 | 说明 |
|------|------|
| **源数据优先（P0）** | 错因 **仅**从 `mistake_memory.json` 重算；History 用于趋势/争议/重复作答；**不读** `bookmark_memory.json` 做错误统计 |
| 只读默认 | 不修改 `mistake_memory`，除非用户另行要求复习打卡 |
| 不编造 | 统计数字必须来自 JSON 或 `aggregation_rules.md` 重算 |
| 五段必出 | 即使用户数据少，也要输出五节结构并标注数据质量 |
| 快照仅写 | `review_retrospective.json` 为**写后缓存**，供 Study Planner 可选融合；过期时以 `source_revisions` 判定 |

---

## 1. Trigger

| 信号 | 动作 |
|------|------|
| 用户说「复盘」 | 进入本工作流 |
| 「学习总结」「错误模式」「我哪方面最弱」 | 同复盘（全量） |
| 「这周复盘」 | `time_window_days = 7` |

**不与 Study Planner 冲突**：用户说「今天学什么」→ Study Planner；「复盘」→ 本工作流。

---

## 2. Load Memory

按顺序读取：

```
memory/data/mistake_memory.json          // 错因分析（必读）
memory/data/question_history.json        // 趋势、争议、重复作答
memory/data/weak_points.json
memory/data/learning_state.json
```

**复盘不读取**：`memory/data/bookmark_memory.json`（收藏不参与错误统计）。  
**禁止作为复盘输入**：`memory/data/review_retrospective.json`。

解析失败 → 告知用户 memory 文件缺失或 JSON 无效，结束。

可选：`memory/user_profile.md`（`exam_target_date`、`current_study_stage`）

---

## 3. Classify & Aggregate

执行 `modules/review_coach/aggregation_rules.md`：

```
Step 3.1  记录分类（confirmed_wrong / disputed / bookmarked / needs_review）
Step 3.2  knowledge_domain_ranking
Step 3.3  error_type_analysis
Step 3.4  weak_points_snapshot（优先 weak_points.json，否则计算）
Step 3.5  pending_review_mistakes（review_status 非 mastered/ignored）
Step 3.6  learning_suggestions（规则模板）
Step 3.7  next_training（主 + 备选）
Step 3.8  data_quality 判定
```

---

## 4. Generate User Output

按 `output_contract.md` §4 渲染 Markdown 五节。

**话术要求**：

- 用「你」称呼用户
- 争议题单独一句说明，不记入错因排名
- 收藏题在复盘中 **不出现** 于错因排名；考前巩固交 Study Planner

---

## 5. Data Handoff

在文末输出 `REVIEW_RETROSPECTIVE` JSON 块（完整结构见 `output_contract.md` §2）。

---

## 6. Optional Persist（快照缓存）

复盘**完成聚合之后**（Step 3–4），可将本次 `REVIEW_RETROSPECTIVE_OUTPUT` 写入快照文件。**写入前内容必须已与当前 `mistake_memory.json` 一致**（刚由 Step 3 生成）。

| 文件 | 内容 |
|------|------|
| `memory/data/review_retrospective.json` | `record_kind: review_snapshot` + `source_revisions` + `last_retrospective` |
| `memory/data/learning_state.json` | 追加 session 事件；`stats.total_reviews += 1` |

快照信封字段（P0）：

| 字段 | 说明 |
|------|------|
| `record_kind` | 固定 `review_snapshot` |
| `snapshot_status` | `current` / `stale`（源文件 `updated_at` 新于 `source_revisions` 时为 `stale`） |
| `source_revisions` | 复制 `mistake_memory.updated_at`、`question_history.updated_at` 等 |
| `last_retrospective` | 与当次 REVIEW_RETROSPECTIVE JSON 同构 |

```json
{
  "event": "retrospective_completed",
  "source_module": "review_coach",
  "data_quality": "partial",
  "timestamp": "ISO8601"
}
```

---

## 7. 工作流总览

```
用户：「复盘」
        │
        ▼
  Load memory/data/*
  （不含 review_retrospective.json）
        │
        ▼
  aggregation_rules.md   ← 始终以最新 mistake_memory 为准
        │
        ▼
  Markdown 五段报告
        │
        ▼
  REVIEW_RETROSPECTIVE JSON
        │
        ▼
  [可选] 覆盖写入 review_retrospective.json（快照 + source_revisions）
```

---

## 8. 与其他模块关系

| 模块 | 关系 |
|------|------|
| Mistake Coach | 上游写入 memory |
| Question Coach | 复盘后可推荐「针对薄弱域再做题」 |
| Study Planner | 可消费 `next_training` 生成计划项（P1） |
| Training Coach | `knowledge_learning` 类型训练的执行方 |

---

## 9. P1 扩展（不在本 MVP）

- 单题复习引导 + `review_count++`
- `ReviewSession` 对齐 `database/learning_schema.md` §5
- 间隔重复算法 `scheduled_review_at`
