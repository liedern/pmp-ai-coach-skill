# Document Input（文档输入）

> **用途**：定义 PDF、Word、Excel、Markdown 等文档类学习资料的输入与路由。
>
> **下游**：`workflows/material_processing.md`

---

## 1. 支持格式

| 格式 | 扩展名 | 预处理 |
|------|--------|--------|
| PDF | `.pdf` | 文本提取；扫描件 → OCR |
| Word | `.docx`, `.doc` | 解析标题、表格、列表 |
| Excel | `.xlsx`, `.csv` | 表头识别，行记录化 |
| Markdown | `.md` | 直接解析 |

---

## 2. 识别信号

- 文件附件或用户说明「这份讲义」「导入题库」
- 无题干选项结构，或明确为教材/考纲/词汇表

---

## 3. 处理流程

```
PDF / Word / Excel / Markdown
      │
      ▼
input_type = document
      │
      ▼
解析 + 元数据（file_name, source_label, pages）
      │
      ▼
Material Processing Workflow
      │
      ├─ exam_material      → knowledge/exam/
      ├─ pmbok_knowledge    → knowledge/pmbok/
      ├─ course_notes       → knowledge/decision_framework/
      ├─ terminology        → knowledge/language/
      ├─ question_bank      → examples/questions/
      └─ wrong_questions    → memory/mistake_memory（映射 schema）
      │
      ▼
knowledge/source/imported_materials.md（索引）
```

---

## 4. output_payload 字段

| 字段 | 说明 |
|------|------|
| `input_type` | `document` |
| `file_ref` | 存储引用 |
| `parsed_text` | 提取正文 |
| `page_map` | 页码映射（可选） |
| `primary_type` | Material Classification 结果 |
| `confidence` | 解析置信度 |
