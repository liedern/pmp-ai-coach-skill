# Memory — MVP 运行时存储

> **版本**：`mvp-1`  
> **定位**：Agent 跨会话可读写的轻量记忆层，映射 `database/` Schema，**不接数据库**。  
> **写入方**：Mistake Coach（主）、Review Coach / Study Planner（后续）

---

## 1. 第一版闭环

```
Question Coach
      │ QUESTION_OUTPUT
      ▼
Mistake Coach
      │ 入库决策 + Mistake 记录
      ▼
memory/data/
      ├── mistake_memory.json    ← 错题明细
      ├── weak_points.json       ← 薄弱知识点聚合
      └── learning_state.json    ← 学习状态 + 高频错误模式
              │
              ▼
      Review Coach（用户：「复盘」）
              │
              └── review_retrospective.json  ← 最近一次复盘快照（可选）
```

---

## 2.1 Review Coach 读取约定

| 触发词 | 工作流 | 写入 |
|--------|--------|------|
| 复盘 | `workflows/review_retrospective.md` | 可选 `review_retrospective.json`、`learning_state` 事件 |

---

## 2. 目录结构

| 路径 | 说明 |
|------|------|
| `memory/*.md` | Schema 定义与 Agent 读写规则（不变） |
| `memory/data/mistake_memory.json` | 运行时错题库 |
| `memory/data/weak_points.json` | 运行时薄弱点 |
| `memory/data/learning_state.json` | 运行时学习状态 |
| `memory/data/examples/` | 填充后的参考示例（只读，不覆盖运行时） |
| `memory/data/review_retrospective.json` | Review Coach 最近一次复盘 JSON 快照 |

**约定**：

- `data/*.json` 为 Agent **运行时读写**文件；初始为空模板
- `data/examples/` 为验证用样例，展示一条完整闭环后的状态
- 单用户 MVP：`user_id` 固定为 `default_user`

---

## 3. 写入时机

| 事件 | 更新文件 |
|------|----------|
| Mistake Coach 入库成功 | `mistake_memory.json` |
| 入库后自动聚合 | `weak_points.json` |
| 每次学习/错题事件 | `learning_state.json` |
| 用户更新掌握度 | `mistake_memory.json` + `learning_state.json` |
| 用户说「复盘」 | `review_retrospective.json` + `learning_state`（`retrospective_completed`） |

Question Coach **不直接写** Memory；必须经过 Mistake Coach。

---

## 4. 文件 Schema 摘要

### 4.1 mistake_memory.json

```json
{
  "version": "mvp-1",
  "user_id": "default_user",
  "mistakes": [],
  "updated_at": null
}
```

每条 `mistakes[]` 对齐 `memory/mistake_memory.md` §2 + `database/mistake_schema.md`。

### 4.2 weak_points.json

```json
{
  "version": "mvp-1",
  "user_id": "default_user",
  "weak_points": [],
  "last_aggregated_at": null
}
```

每条 `weak_points[]` 对齐 `memory/weak_points.md` §2 + `database/learning_schema.md` WeakPoint。

### 4.3 learning_state.json

```json
{
  "version": "mvp-1",
  "user_id": "default_user",
  "current_study_stage": "practice",
  "error_patterns": {},
  "sessions": [],
  "stats": {},
  "updated_at": null
}
```

| 字段 | 说明 |
|------|------|
| `error_patterns` | 高频错误模式，键为 `mistake_type` 或 `knowledge_point` |
| `sessions` | 学习事件时间线 |
| `stats` | 累计统计（做题数、错题数、正确率等） |

---

## 5. 聚合规则（weak_points）

Mistake Coach 每次入库后执行：

```
1. 读取 mistake_memory.mistakes
2. 按 knowledge_point 主标签分组
3. error_count = 组内 sum(repeated_count)
4. dominant_mistake_type = 组内 mistake_type 众数
5. error_trend:
   - 首次出现 → new
   - 近 7 天有新增 → rising
   - 否则 stable
6. priority_score = min(100, error_count * 15 + max_repeated_count * 10)
7. 写回 weak_points.json
```

---

## 6. 错误模式统计（error_patterns）

`learning_state.json` 中 `error_patterns` 结构：

```json
{
  "by_mistake_type": {
    "scenario_judgment_error": { "count": 3, "last_at": "2026-07-29T18:05:00+08:00" },
    "concept_confusion": { "count": 1, "last_at": "2026-07-27T10:00:00+08:00" }
  },
  "by_knowledge_point": {
    "冲突管理": { "count": 2, "dominant_type": "scenario_judgment_error" },
    "变更管理": { "count": 1, "dominant_type": "process_order_error" }
  },
  "top_patterns": [
    {
      "pattern": "冲突类题目倾向过度升级",
      "mistake_type": "scenario_judgment_error",
      "knowledge_points": ["冲突管理"],
      "occurrence_count": 2
    }
  ]
}
```

`top_patterns` 在同类错误 ≥ 2 次时生成，供 Agent 个性化提示。

---

## 7. Agent 读写规则

| 操作 | 规则 |
|------|------|
| 读取 | 讲解前可读 `mistake_memory`、`weak_points` 做个性化 |
| 写入 | 仅 Mistake Coach 流程写入；不编造未发生的错题 |
| 合并 | 同 `question_id` 更新，不重复追加 |
| 备份 | MVP 无版本控制；用户可手动备份 `data/` 目录 |

---

## 8. 与 database/ 映射

| Memory 文件 | Database Schema |
|-------------|-----------------|
| `mistake_memory.json` → `mistakes[]` | `mistake_schema.md` |
| `weak_points.json` → `weak_points[]` | `learning_schema.md` WeakPoint |
| `learning_state.json` → `sessions[]` | `learning_schema.md` LearningProgress |

未来 Web 产品：将 `data/*.json` 替换为 API 调用，字段名保持不变。

---

## 9. 验证方法

1. 将 `modules/question_coach/examples/sample_output.json` 作为 Question Coach 输出
2. 按 `modules/mistake_coach/decision_rules.md` 执行入库
3. 对比 `memory/data/examples/` 下三个 JSON 的预期结构
4. 确认 `wrong_type` 映射、`repeated_count`、`error_patterns` 计数正确
