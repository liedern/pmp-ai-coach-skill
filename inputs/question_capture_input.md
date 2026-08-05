# Question Capture Input（单题采集输入）

> **用途**：定义**最少字段**下的单题录入格式，供真实刷题、截图、事后补录使用。  
> **下游**：`workflows/question_capture.md` → `workflows/question_analysis.md`

---

## 1. 输入类型

| input_type | 说明 |
|------------|------|
| `question_capture` | 单题采集（文字 / 截图 / 混合） |
| 兼容 | 识别为 `question` 或 `image` 时，**同样**先走 Capture 再 Analysis |

---

## 2. 最少输入（按优先级）

用户**不必**按固定模板；Agent 从消息中提取。推荐极简形式：

| 优先级 | 用户提供 | 档位 |
|--------|----------|------|
| 1 | 截图（含选项） | L0 |
| 2 | +「我选 X」 | L1 |
| 3 | +「正确 Y」或解析里的标答 | L2 |
| 4 | + 解析全文 | L3 |

**一行命令**（可选）：

```text
录入
```

表示进入单题采集；后续可只发图或只发文字。

---

## 3. 识别信号

- 动词：`录入`、`记一道`、`刷题记录`、`练习 x/180`
- 答案：`我选 A`、`选B`、`用户答案：C`
- 标答：`正确 D`、`答案：B`、`官方解析`
- 收藏：`收藏`、`已加入收藏`、`加入重点题`
- 跳过持久化：`只要讲解`、`不要保存`

---

## 4. output_payload（Capture 阶段）

| 字段 | 说明 |
|------|------|
| `capture_level` | `L0`–`L3` |
| `question_id` | 新建或匹配 |
| `question_text` | 题干 |
| `options` | 选项对象 |
| `user_answer` | 可 null |
| `correct_answer` | 可 null |
| `official_explanation` | 可 null |
| `source` / `exam_set` | 可 null |
| `ocr_text` | 截图时 |
| `data_routing` | `write_history` / `write_mistake` / `write_bookmark` |

完整契约见 `workflows/question_capture.md` §8 `CAPTURE_RECORD`。

---

## 5. 路由

```
question_capture_input
      │
      ▼
workflows/question_capture.md
      │
      ▼
workflows/question_analysis.md
      │
      ├─ memory/data/question_history.json
      ├─ memory/data/mistake_memory.json（答错）
      └─ memory/data/bookmark_memory.json（仅收藏）
```
