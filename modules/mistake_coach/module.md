# Mistake Coach — MVP 模块实现

> **所属产品**：PMP AI Coach（单一主 Skill）  
> **模块类型**：专业能力模块 | **非独立 Skill**  
> **上游**：Question Coach `QUESTION_OUTPUT`  
> **下游**：`memory/data/*`（MVP 无数据库）

---

## 1. 模块定位

Mistake Coach 承接 Question Coach 的结构化输出，**答错即自动入库**（`user_answer` ≠ `correct_answer`），无需用户说「保存错题」：

```
QUESTION_OUTPUT（DATA_HANDOFF）
        │
        ▼
入库决策（P0：是否真实错题 → 是否写入错题库）
        │
        ├─ 否 → 输出决策说明，结束
        │
        └─ 是 → 生成 Mistake 记录
                    │
                    ▼
            更新 memory/data/
            ├── mistake_memory.json
            ├── weak_points.json
            └── learning_state.json
```

**不负责**：单题 PMP 推理与讲解（→ Question Coach）、复习执行（→ Review Coach）。

---

## 2. 输入契约

### 2.1 首选输入

Question Coach 的 `QUESTION_OUTPUT`（见 `modules/question_coach/output_contract.md`）。

### 2.2 补充输入

| 来源 | 用途 |
|------|------|
| `memory/data/mistake_memory.json` | 去重、`repeated_count` 累加 |
| `memory/data/weak_points.json` | 薄弱点趋势 |
| 用户显式指令 | 「不要保存」→ `force_skip`；「加入错题本」**仅在答错时**生效 |

### 2.3 输入 JSON（`MISTAKE_INPUT`）

```json
{
  "source_module": "question_coach",
  "question_output": { },
  "user_override": {
    "force_save": null,
    "force_skip": false
  }
}
```

| 字段 | 说明 |
|------|------|
| `question_output` | 完整 `QUESTION_OUTPUT` |
| `user_override.force_save` | 已废弃绕过 P0；答对时不得入库 |
| `user_override.force_skip` | `true` 时强制不入库 |

---

## 3. 执行流程

```
0. Real Mistake Gate  ← decision_rules.md §0（P0）
1. Validate Input       ← 校验 QUESTION_OUTPUT 必填字段
2. Save Decision        ← decision_rules.md §1
3. Dedup Check          ← 对比 mistake_memory.json
4. Build Mistake Record ← 对齐 mistake_schema.md
5. Update Memory        ← 写入三个 JSON 文件
6. Output Handoff       ← MISTAKE_OUTPUT
```

详细步骤见 `workflows/mistake_classification.md`。

---

## 4. 输出契约（`MISTAKE_OUTPUT`）

````markdown
<!-- MISTAKE_HANDOFF:BEGIN -->
```json
{ ... }
```
<!-- MISTAKE_HANDOFF:END -->
````

### 4.1 顶层结构

```json
{
  "module": "mistake_coach",
  "version": "mvp-1.1",
  "save_decision": {
    "should_save": true,
    "reason": "真实错题：user_answer=A, correct_answer=B",
    "is_real_mistake": true
  },
  "mistake_record": null,
  "memory_updates": {
    "mistake_memory": "created | updated | skipped",
    "weak_points": "reaggregated | skipped",
    "learning_state": "appended | skipped"
  },
  "pattern_insight": null,
  "created_at": "2026-07-29T18:05:00+08:00"
}
```

### 4.2 `mistake_record`

入库时生成，**字段对齐** `database/mistake_schema.md`。完整示例见 `examples/sample_mistake_record.json`。

MVP 真实错题必填字段（对齐 `memory_record_contract.md`，**仅**写入 `mistakes[]` 核心九项）：

| 字段 | 说明 |
|------|------|
| `mistake_id` | UUID 或 `mistake_{timestamp}` |
| `question_id` | 内容指纹或 UUID |
| `user_answer` | 用户答案 |
| `correct_answer` | 正确答案 |
| `error_type` | §`decision_rules.md` 3.1 产品五项 |
| `error_reason` | 错因说明 |
| `knowledge_point` | 知识点列表 |
| `review_status` | 初始 `new` |

可选：`exam_domain`。**禁止新写** `mistake_type`、`mistake_reason`、`eco_domain`（见 `schema_overview.md` §1.1）。

题干快照、选项、`wrong_type` 等见 `database/mistake_schema.md` 关系型扩展；**对错 / 争议 / 收藏** 见 `question_history.json` / `bookmark_memory.json`。

### 4.3 `pattern_insight`（可选）

当检测到重复错误或高频错因时输出：

```json
{
  "is_repeat": true,
  "repeated_count": 2,
  "dominant_error_type": "scenario_judgment_error",
  "related_knowledge_points": ["冲突管理"],
  "message": "你在「冲突管理」类题目中第 2 次出现场景判断错误"
}
```

---

## 5. Memory 写入规则

| 文件 | 触发 | 写入内容 |
|------|------|----------|
| `memory/data/mistake_memory.json` | `should_save=true` | 新增或更新错题条目 |
| `memory/data/weak_points.json` | 每次入库后 | 按 `knowledge_points` 重新聚合 |
| `memory/data/learning_state.json` | 每次入库后 | 追加学习事件、更新错误模式统计 |

写入格式见 `memory/README.md`。

---

## 6. Agent 约束

| 约束 | 说明 |
|------|------|
| P0 真实错题 | `user_answer` ≠ `correct_answer` 才入库；答对、收藏、做对不确信均不入库 |
| 不重复创建 | 同一 `question_id` 更新已有记录，`repeated_count += 1` |
| 不编造 | 无 `user_answer` 或缺 `error_type` / `error_reason` 不入库 |
| 映射一致 | `error_type` 产品五项；读时兼容 `mistake_type`；可选 `wrong_type` 映射 §3.3 |
| 不连数据库 | MVP 仅写 `memory/data/*.json` |

---

## 7. 文件索引

| 文件 | 说明 |
|------|------|
| `module.md` | 本文件 |
| `decision_rules.md` | P0 门槛、入库决策矩阵与字段映射 |
| `memory_record_contract.md` | `mistake_memory.json` 单条写入契约 |
| `examples/sample_mistake_record.json` | Mistake 记录示例 |
| `examples/sample_handoff.json` | 完整 MISTAKE_OUTPUT 示例 |
| `workflows/mistake_classification.md` | 执行步骤 |
| `database/mistake_schema.md` | 持久化 Schema 权威定义 |
