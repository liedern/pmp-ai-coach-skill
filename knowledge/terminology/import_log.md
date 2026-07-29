# Material Processing Report — Terminology Import

| 字段 | 值 |
|------|-----|
| processing_id | `mp-terminology-20260728-001` |
| source_path | `source_materials/terminology/PMP中英文词组翻译.pdf` |
| source_label | PMP 中英文词组翻译（高频缩写与中英文对照完整版） |
| input_type | pdf（图像页，OCR 识别） |
| primary_type | terminology |
| units_processed | 36 |
| items_extracted | 约 380+ 词条 |
| items_stored | 见 `pmp_glossary.md` 及各 `knowledge/language/` 文件 |
| need_review_count | 2（FOW、PF 为非标准缩写，已标注不建议使用） |
| stored_paths | `knowledge/terminology/pmp_glossary.md`, `knowledge/language/*.md` |
| processed_at | 2026-07-28T14:30:00+08:00 |

## Need Review 清单

| 字段 | 原因 |
|------|------|
| FOW | 源 PDF 标注为非标准写法，不建议使用 |
| PF | 源 PDF 标注为不规范 Float 缩写，标准术语为 TF / FF |

## 写入摘要

- `knowledge/terminology/pmp_glossary.md`：全量章节术语表（【事实】译文来自 PDF）
- `knowledge/language/pmp_terms.md`：高频术语详解（含考试语境【推测】补充）
- `knowledge/language/confusing_terms.md`：易混概念扩展
- `knowledge/language/synonym_mapping.md`：同义词与旧称映射
- `knowledge/language/keyword_mapping.md`：按章节关键词索引

<!-- MATERIAL_HANDOFF:BEGIN -->
```json
{
  "processing_id": "mp-terminology-20260728-001",
  "source_path": "source_materials/terminology/PMP中英文词组翻译.pdf",
  "primary_type": "terminology",
  "items_stored": 380,
  "target_paths": [
    "knowledge/terminology/pmp_glossary.md",
    "knowledge/language/pmp_terms.md",
    "knowledge/language/confusing_terms.md",
    "knowledge/language/synonym_mapping.md",
    "knowledge/language/keyword_mapping.md"
  ],
  "need_review_items": ["FOW", "PF"]
}
```
<!-- MATERIAL_HANDOFF:END -->
