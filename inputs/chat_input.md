# Chat Input（对话输入）

> **用途**：定义自由对话类输入的意图识别与 Workflow 路由。
>
> **下游**：Question Answer / Learning Plan / Question Analysis（附带题目时）

---

## 1. 输入形态

| 形态 | 示例 |
|------|------|
| 概念答疑 | 「Risk 和 Issue 有什么区别？」 |
| 学习规划 | 「帮我制定两周学习计划」 |
| 进度汇报 | 「今天做了 30 道题」 |
| 薄弱点查询 | 「我哪方面最弱？」 |
| 混合消息 | 对话中粘贴题目 |

---

## 2. 意图分类

| 意图 | 路由 |
|------|------|
| `teach` | Question Answer Flow → `skill.md` Teaching Mode |
| `plan` | Learning Plan Flow → `workflows/study_plan.md` |
| `analyze_question` | 若含题目结构 → `question_analysis` |
| `review_progress` | 读取 `memory/learning_progress`、`weak_points` |
| `save_mistake` | 「加入错题本」→ `question_analysis`（**仅答错已自动入库**；答对则说明不入库） |
| `process_material` | 「整理这份资料」→ material_processing |
| `general` | skill.md 通用教练回复 |

---

## 3. 处理流程

```
自由对话
      │
      ▼
input_type = chat
      │
      ▼
意图分类（teach | plan | analyze | …）
      │
      ├─ teach     → Knowledge 检索 + Teaching Mode
      ├─ plan      → Memory 读取 + study_plan
      ├─ analyze   → question_analysis（若含题）
      └─ progress  → memory 聚合报告
      │
      ▼
生成回答 + 可选更新 Memory
```

---

## 4. Memory 读写

| 意图 | 读取 | 写入 |
|------|------|------|
| `plan` | user_profile, weak_points, mistake_memory | learning_progress.next_plan |
| `review_progress` | learning_progress, weak_points | — |
| 日终总结 | — | learning_progress 新条目 |
| `teach` | weak_points（个性化） | 可选 mastered_content |

---

## 5. output_payload 字段

| 字段 | 说明 |
|------|------|
| `input_type` | `chat` |
| `raw_message` | 用户原文 |
| `intent` | 分类结果 |
| `embedded_question` | 消息内嵌题目对象（若有） |
| `confidence` | 意图置信度 |
