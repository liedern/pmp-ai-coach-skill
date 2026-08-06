# Question Schema

> 共享题目实体 + 用户作答交互（History / Bookmark）数据模型。  
> **消费模块**：Question Coach（写 Question / History）；Mistake Coach（读 Question）；Bookmark 由用户主动触发写入。

---

## 0. 三分流原则（P0）

同一道题、同一次作答，可同时产生 **最多三类** 记录，职责不可混淆：

| 存储 | 实体 | 触发 | 是否答错 |
|------|------|------|----------|
| Mistake Memory | `Mistake` | `user_answer` ≠ **`adjudication_answer`** → **自动**（v0.1.1） | **必须答错**（相对判题基准） |
| Bookmark Memory | `Bookmark` | 用户主动「收藏」 | 对错均可 |
| Question History | `QuestionHistory` | 每次做题结束 → **自动** | 对错均记 |

```
用户做题
   │
   ├─► QuestionHistory（始终）
   ├─► Mistake（仅答错，自动）
   └─► Bookmark（仅主动收藏）
```

---

## 1. Purpose — Question

`Question` 表示题库中的一道标准题，与用户的错题 / 收藏 / 历史分离：

```
Question (1) ──< (N) Mistake
Question (1) ──< (N) Bookmark
Question (1) ──< (N) QuestionHistory
```

| 用途 | 说明 |
|------|------|
| 题目归一化 | 同一题多次做 → 同一 `question_id` |
| 数据分层 | 题目正文存 `Question`；对错/错因存 Mistake；主动关注存 Bookmark；轨迹存 History |
| Web 扩展 | 多用户共享同一 `Question` 记录 |

### MVP 写入流程

Question Coach 分析完成后：

1. 创建或匹配 `Question`（按题干 + 选项）
2. **始终**追加 `QuestionHistory`
3. 若 `user_answer` ≠ **`adjudication_answer`**（v0.1.1；legacy 无 AEL 时 = `correct_answer`）→ **自动**触发 Mistake Coach
4. 若用户主动收藏 → 写 `Bookmark`（独立于 Mistake）

---

## 2. Question 字段（MVP）

| 字段名称 | 类型建议 | 是否必须 | 用途 |
|----------|----------|----------|------|
| `question_id` | UUID | **是** | 主键 |
| `question_text` | TEXT | **是** | 题干全文 |
| `options` | JSON | **是** | `{"A":"...","B":"...","C":"...","D":"..."}` |
| `correct_answer` | VARCHAR(32) | 否 | 正确答案；未知为 `null` |
| `source` | VARCHAR(255) | 否 | 题目来源（机构、PMBOK、自编等）；与 `question_source` 可二选一或同值 |
| `question_source` | VARCHAR(255) | 否 | 与 `source` 同义，Question Coach handoff 常用键名 |
| `paper_id` | VARCHAR(128) | 否 | 套卷/试卷标识（如「模拟卷第 2 套」）；可与 History 的 `exam_set` 对齐 |
| `knowledge_point` | VARCHAR(255) | 否 | 主知识点标签 |
| `created_time` | TIMESTAMPTZ | **是** | 入库时间，ISO 8601 |

**向后兼容**：历史数据可能仅有 `source` 而无 `question_source` / `paper_id`；读取时合并 `source` ← `question_source`，`paper_id` 缺失时可用 `exam_set`（History）补全。

---

## 3. QuestionHistory（做题历史）

> Memory 文件：`memory/data/question_history.json`  
> **所有做过的题都记录**，无论对错、是否收藏。

### 3.1 字段

| 字段 | 类型 | 必须 | 说明 |
|------|------|:----:|------|
| `history_id` | UUID | 是 | 主键 |
| `question_id` | UUID | 是 | 关联 Question |
| `user_id` | UUID | 是 | 多用户隔离 |
| `user_answer` | VARCHAR(32) | 否 | 用户答案；未作答为 `null` |
| `correct_answer` | VARCHAR(32) | 否 | 当次 **判题基准**（= `adjudication_answer` 快照）；v0.1.0 历史条可能仅为题库标答 |
| `answer_evaluation` | JSON | 否 | v0.1.1：`platform_answer`、`coach_answer`、`answer_confidence`、`answer_status`、`answer_disputed`、`adjudication_answer`、`ael_version` |
| `result` | ENUM | 是 | `correct` / `wrong` / `unknown` / `skipped`（**唯一**对错字段；Mistake 不写 `is_correct`） |
| `answer_disputed` | BOOLEAN | 否 | 答案/题库版本争议；复盘 `disputed` 分类读此字段 |
| `correct_answer_note` | TEXT | 否 | 争议说明或多版本答案备注 |
| `source` | VARCHAR(255) | 否 | 题目来源 |
| `exam_set` | VARCHAR(255) | 否 | 套卷编号（如「练习 166/180」） |
| `created_at` | TIMESTAMPTZ | 是 | 本次作答时间 |

### 3.2 用途

- 学习统计、做题数量、正确率
- 学习轨迹时间轴
- **不**直接驱动 Review Coach 错因分析（Review 只读 Mistake）

### 3.3 写入规则

| 规则 | 说明 |
|------|------|
| 每次 Question Coach 完成且存在 `user_answer` | 必写一条 History |
| 仅讲解、无作答 | 可不写，或 `result=skipped` |
| 答错 | History `result=wrong` **且** 另写 Mistake |
| 答对 | History `result=correct`；**不**写 Mistake |
| 收藏 | History 仍记本次 `result`；**另写** `bookmark_memory.json`（收藏行为不在 History 重复存 tag） |

### 3.4 最小 JSON

```json
{
  "history_id": "hist_evm_spi_001",
  "question_id": "q_hybrid_sprint3_ev_pv_001",
  "user_id": "default_user",
  "user_answer": "A",
  "correct_answer": "D",
  "result": "wrong",
  "answer_evaluation": {
    "platform_answer": "D",
    "coach_answer": "D",
    "answer_confidence": "high",
    "answer_status": "aligned",
    "answer_disputed": false,
    "adjudication_answer": "D",
    "ael_version": "0.1.1"
  },
  "source": "培训机构题库",
  "exam_set": "练习 166/180",
  "created_at": "2026-07-31T00:59:00+08:00"
}
```

争议作答示例（`answer_disputed=true`；**相对 adjudication 仍可能答错**；争议且用户=Coach 时不写 Mistake）：

```json
{
  "history_id": "hist_it_deliverable_stakeholder_concern_001",
  "question_id": "q_it_deliverable_stakeholder_concern_001",
  "user_id": "default_user",
  "user_answer": "D",
  "correct_answer": "B",
  "result": "wrong",
  "answer_disputed": true,
  "correct_answer_note": "多版本题库答案不一致",
  "created_at": "2026-07-30T20:12:00+08:00"
}
```

---

## 4. Bookmark（收藏题）

> Memory 文件：`memory/data/bookmark_memory.json`  
> **仅用户主动行为**写入。收藏 ≠ 错题。

### 4.1 字段

| 字段 | 类型 | 必须 | 说明 |
|------|------|:----:|------|
| `bookmark_id` | UUID | 是 | 主键 |
| `question_id` | UUID | 是 | 关联 Question |
| `user_id` | UUID | 是 | 多用户隔离 |
| `bookmark_reason` | TEXT | 否 | 如「经典 EVM」「考前必看」 |
| `tags` | JSON / TEXT[] | 否 | 用户或系统标签 |
| `created_at` | TIMESTAMPTZ | 是 | 收藏时间 |

可选扩展（非最小集）：`question_text` 快照、`source`、`exam_set`。

### 4.2 触发（仅主动）

| 用户行为 | 是否写 Bookmark |
|----------|-----------------|
| 点击收藏 / 「收藏这题」/ 「加入我的重点题」 | **是** |
| 答错自动入库 | **否**（写 Mistake，不写 Bookmark） |
| 答对但未说收藏 | **否** |

### 4.3 用途

- 用户主动关注、高频考点收藏
- 考前快速复习
- Study Planner **辅助**推荐（「复习收藏重点题」）
- **Review Coach 错因分析不得读取 Bookmark 当错误**

### 4.4 最小 JSON

```json
{
  "bookmark_id": "bm_evm_classic_001",
  "question_id": "q_hybrid_sprint3_ev_pv_001",
  "user_id": "default_user",
  "bookmark_reason": "经典 SPI/Fast Track 考点",
  "tags": ["EVM", "进度压缩", "hybrid"],
  "created_at": "2026-07-31T01:00:00+08:00"
}
```

### 4.5 与 Mistake 对照示例

经典 EVM 题，用户 **答对** 且 **收藏**：

| 存储 | 结果 |
|------|------|
| Question History | ✅ `result=correct` |
| Bookmark Memory | ✅ |
| Mistake Memory | ❌ |

---

## 5. 与 Mistake 的关联

| 规则 | 说明 |
|------|------|
| 外键 | `Mistake.question_id` → `Question.question_id` |
| MVP | 写 Mistake 前须先创建/匹配 Question |
| 去重 | 同一题干 + 选项 → 复用 `question_id` |

---

## 6. 与 DATA_HANDOFF 映射

| DATA_HANDOFF | 目标 |
|--------------|------|
| `question_text` / `options` / `correct_answer` / `source` / `question_source` / `paper_id` | → Question |
| `user_answer` + `correct_answer` + `result` | → QuestionHistory |
| `error_type` + `error_reason`（答错时） | → Mistake（经 Mistake Coach） |
| 用户收藏意图 | → Bookmark |

---

## 7. Future Extension

`content_hash`、共享题库审核、`bookmark.priority`、History 分页索引等 — Web 产品阶段按需扩展。
