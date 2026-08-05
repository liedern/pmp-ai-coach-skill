# 示例：答错题分析（Question Coach 输出摘要）

> 对应工作流：[`workflows/question_capture.md`](../workflows/question_capture.md) → [`workflows/question_analysis.md`](../workflows/question_analysis.md)  
> 数据路由：History ✅ · Mistake ✅ 自动 · Bookmark ❌  
> 数据集：**Example Dataset**（虚构 ID，非运行时 Memory）

---

## 用户输入

题干：项目经理发现项目进度落后，EV=500000，PV=550000，应该采取什么措施？

选项：  
A. Introduce a float  
B. Adjust the budget  
C. Adjust the project timeline  
D. Fast track the project  

**我的答案：A**

---

## Agent 可见输出（结构节选）

### 正确答案

**D — Fast track the project**

EV &lt; PV → SPI &lt; 1，进度落后；应优先考虑**进度压缩**（如 Fast track），而非引入浮动时间挽回已落后进度。

### 用户错因

- **错误类型**（`error_type`）：`concept_confusion`  
- **错误原因**（`error_reason`）：误将「引入浮动时间」当作进度落后时的首选措施。  

### 归档（自动，无需确认）

- **本题已自动加入错题库**  
- **知识点**：挣值管理、SPI、进度压缩、Fast Track  

---

## Memory 写入结果（示意）

**`question_history.json`** — 追加一条：

| 字段 | 值 |
|------|-----|
| `question_id` | `q_example_evm_001` |
| `user_answer` | `A` |
| `correct_answer` | `D` |
| `result` | `wrong` |
| `source` | `Mock PMP Set — Example Dataset` |
| `exam_set` | `Mock Set A / Q12` |
| `answer_time` | ISO 8601 |

**`mistake_memory.json`** — 追加一条（核心字段）：

```json
{
  "mistake_id": "mistake_example_evm_001",
  "question_id": "q_example_evm_001",
  "user_answer": "A",
  "correct_answer": "D",
  "error_type": "concept_confusion",
  "error_reason": "EV<PV 表示进度落后（SPI<1），应优先考虑进度压缩（如 Fast track）。",
  "knowledge_point": ["挣值管理", "SPI", "进度压缩", "Fast Track"],
  "exam_domain": "process",
  "review_status": "new"
}
```

**`bookmark_memory.json`** — 无变化。

完整 JSON 交接见：`modules/question_coach/examples/sample_output.json`、`modules/mistake_coach/examples/sample_mistake_record.json`。
