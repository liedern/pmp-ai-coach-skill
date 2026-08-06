# Image Input（图片输入）

> **用途**：定义题目截图、讲义拍照等图片输入的 OCR 与路由规则。
>
> **下游**：`question_capture` → `question_analysis`（题目图）或 `material_processing`（资料图）

---

## 1. 输入形态

| 形态 | 说明 |
|------|------|
| 单题截图 | 含题干 + 选项 |
| 多题页面 | 需切分或逐题 OCR |
| 讲义/考纲拍照 | 非题目，走资料加工 |
| 手写笔记 | 低置信，易 Need Review |

---

## 2. 处理流程

### 2.1 题目截图（主路径）

```
图片截图
      │
      ▼
input_type = image
      │
      ▼
OCR 识别（未来扩展：ocr_metadata）
      │
      ▼
题目结构化（题干 / 选项 / 置信度）
      │
      ▼
Question Capture Workflow
      │
      ▼
Question Analysis Workflow（须执行 `question_analysis.md` §2.4–§2.5：题干优先、截图两阶段防锚定）
      │
      ▼
Mistake Database / mistake_memory（用户要求保存时）
```

### 2.2 非题目图片

```
图片截图
      │
      ▼
OCR → 判定非题目结构
      │
      ▼
Material Processing Workflow
      │
      ▼
Knowledge Base
```

---

## 3. OCR 质量规则

| 置信度 | 处理 |
|--------|------|
| high | 直接进入 question_analysis |
| medium | 标注【待确认】字段，向用户确认 |
| low | Need Review，不自动判答案 |

---

## 4. output_payload 字段

| 字段 | 说明 |
|------|------|
| `input_type` | `image` |
| `image_ref` | 图片 URI / 附件 ID |
| `ocr_text` | OCR 全文 |
| `ocr_confidence` | 0–1 或 high/medium/low |
| `structured_question` | 解析后的题目对象（若适用） |
| `routing` | `question_analysis` \| `material_processing` |
