# Question Input（题目文字输入）

> **用途**：定义用户以**纯文本**形式提交 PMP 题目时的输入格式、识别规则与路由。
>
> **下游**：`workflows/question_analysis.md`

---

## 1. 输入形态

| 形态 | 说明 |
|------|------|
| 完整题目 | 题干 + 选项 A–D（或 1–4） |
| 题干 + 用户答案 | 含「我选了 X」 |
| 题干 + 答案 + 解析 | 含正确答案与官方解析 |
| 不完整片段 | 仅题干或选项缺失 → 标 Need Review |

---

## 2. 识别信号

- 含选项标记：`A.` `B)` `(A)` `A、` 等
- 用户标注：`我的答案`、`正确答案`、`解析`
- 题号：`Q12`、`第 15 题`

---

## 3. 处理流程

```
题目文字粘贴
      │
      ▼
input_type = question
      │
      ▼
结构化提取（题干 / 选项 / 用户答案 / 正确答案）
      │
      ▼
Question Analysis Workflow
      │
      ├─ 仅讲解 → 输出分析，不写 Memory
      └─ 保存错题 → mistake_memory → [可选] database
```

---

## 4. output_payload 字段

| 字段 | 说明 |
|------|------|
| `input_type` | `question` |
| `question_text` | 题干 |
| `options` | `{"A":"...","B":"..."}` |
| `user_answer` | 可选 |
| `correct_answer` | 可选 |
| `source` | 用户说明或 null |
| `confidence` | high / medium / low |
