# 示例：答对 + 收藏（Question Coach 输出摘要）

> 数据路由：History ✅ · Mistake ❌ · Bookmark ✅（仅当用户说「收藏这题」）  
> 数据集：**Example Dataset**（虚构 ID）

---

## 用户输入

（敏捷价值交付类题目，用户作答 **A**，正确答案 **A**）

分析完成后用户说：**「收藏这题」**

---

## Agent 可见输出（结构节选）

### 正确答案

**A** — 功能提前交付，更多价值被交付（考点：敏捷价值与增量交付）。

### 归档

- **不**出现「是否保存错题」提示（答对不入错题库）。  
- 收藏后：**「已加入收藏」**（写入 Bookmark，**不**写入 Mistake）。

---

## Memory 写入结果（示意）

**`question_history.json`** — 追加一条：

| 字段 | 值 |
|------|-----|
| `question_id` | `q_example_agile_value_001` |
| `user_answer` | `A` |
| `correct_answer` | `A` |
| `result` | `correct` |
| `source` | `Mock PMP Set — Example Dataset` |
| `exam_set` | `Mock Set A / Q08` |
| `answer_time` | ISO 8601 |

**`mistake_memory.json`** — **无新增**（`user_answer === correct_answer`）。

**`bookmark_memory.json`** — 追加一条：

```json
{
  "bookmark_id": "bm_example_agile_value_001",
  "question_id": "q_example_agile_value_001",
  "user_id": "default_user",
  "bookmark_reason": "敏捷价值交付与项目评估（示例）",
  "tags": ["价值交付", "敏捷原则", "增量交付", "Product Owner"],
  "created_at": "ISO 8601"
}
```

---

## 与错题的边界

| 行为 | History | Mistake | Bookmark |
|------|:-------:|:-------:|:--------:|
| 答对 | ✅ | ❌ | 仅用户收藏时 ✅ |
| 答错 | ✅ | ✅ 自动 | 可选 ✅ |

Review Coach **不**把收藏计入错误统计；考前复习收藏题由 Study Planner 作**低权重**辅助任务。
