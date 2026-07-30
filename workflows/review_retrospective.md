# Review Retrospective Workflow（学习复盘）

> 用户触发 **「复盘」** 时执行。  
> **读取** Mistake Coach / Memory 数据，**不**接数据库。  
> **模块契约**：`modules/review_coach/module.md`

---

## 0. 执行原则

| 原则 | 说明 |
|------|------|
| 只读默认 | 不修改 `mistake_memory`，除非用户另行要求复习打卡 |
| 不编造 | 统计数字必须来自 JSON 或 `aggregation_rules.md` 重算 |
| 五段必出 | 即使用户数据少，也要输出五节结构并标注数据质量 |

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
memory/data/mistake_memory.json
memory/data/weak_points.json
memory/data/learning_state.json
```

可选：`memory/user_profile.md`（`exam_target_date`、`current_study_stage`）

解析失败 → 告知用户 memory 文件缺失或 JSON 无效，结束。

---

## 3. Classify & Aggregate

执行 `modules/review_coach/aggregation_rules.md`：

```
Step 3.1  记录分类（confirmed_wrong / disputed / bookmarked / needs_review）
Step 3.2  knowledge_domain_ranking
Step 3.3  mistake_type_analysis
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
- 收藏题表述为「待巩固」而非「错题」

---

## 5. Data Handoff

在文末输出 `REVIEW_RETROSPECTIVE` JSON 块（完整结构见 `output_contract.md` §2）。

---

## 6. Optional Persist

若用户未拒绝持久化（默认 MVP 可写入）：

| 文件 | 内容 |
|------|------|
| `memory/data/review_retrospective.json` | 本次 JSON 快照 + `generated_at` |
| `memory/data/learning_state.json` | 追加 session 事件；`stats.total_reviews += 1` |

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
        │
        ▼
  aggregation_rules.md
        │
        ▼
  Markdown 五段报告
        │
        ▼
  REVIEW_RETROSPECTIVE JSON
        │
        ▼
  [可选] review_retrospective.json
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
