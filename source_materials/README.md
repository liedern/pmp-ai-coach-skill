# Source Materials（原始资料库）

> 本目录用于保存**未经加工**的 PMP 学习原始资料。
>
> **重要**：此处文件不得被 Agent 直接当作正式知识引用。所有资料必须经过 [`workflows/material_processing.md`](../workflows/material_processing.md) 加工、质检通过后，方可写入 [`knowledge/`](../knowledge/) 目录。

---

## 目录说明

| 子目录 | 用途 | 典型内容 | 加工后目标路径 |
|--------|------|----------|----------------|
| `exam/` | PMP 考试大纲、考纲类官方或培训机构资料 | ECO、Exam Content Outline、报考说明、题型说明 | `knowledge/exam/` |
| `textbook/` | PMBOK 及系统教材 | PMBOK PDF、辅导书扫描件、章节笔记 | `knowledge/pmbok/`、`knowledge/agile/` |
| `course/` | 培训课程笔记、讲师讲义 | 视频课文字稿、冲刺讲义、口诀总结 | `knowledge/decision_framework/`、`knowledge/course/`（若扩展） |
| `terminology/` | 中英文词汇、缩写表 | 词汇表 Excel、术语对照 PDF | `knowledge/language/`、`knowledge/terminology/` |
| `question_bank/` | 题库、模拟题原始文件 | 套卷 PDF、带解析 Word、题库 CSV | `examples/questions/`；错题类 → `memory/` / `database/` |

---

## 使用流程

```
原始文件放入 source_materials/{分类}/
        │
        ▼
触发 Material Processing Workflow
        │
        ├─ 分类 → 提取 → 结构化 → 质检
        └─ Need Review 项需人工确认
        │
        ▼
写入 knowledge/（或 examples/、memory/）
        │
        ▼
在 knowledge/source/imported_materials.md 登记索引
```

---

## 规范

1. **保留原文件**：加工后不删除原始文件，便于溯源与版本对比。
2. **命名建议**：`{来源}_{主题}_{版本或日期}.{扩展名}`，如 `PMI_ECO_2021.pdf`。
3. **版权**：仅存放用户有权使用的学习资料；敏感材料勿提交公开仓库。
4. **不直接引用**：Agent 推理应读取 `knowledge/`，而非本目录原文。
5. **重复导入**：同一资料更新时保留旧版或注明版本，避免覆盖丢失。

---

## 与项目其他目录的关系

| 目录 | 关系 |
|------|------|
| `source_materials/` | 原始输入（本目录） |
| `workflows/material_processing.md` | 加工流水线定义 |
| `knowledge/` | 结构化、可引用的正式知识库 |
| `knowledge/source/imported_materials.md` | 已导入材料的索引与元数据 |
| `inputs/document_input.md` | 文档类输入的路由说明 |
