# Imported Materials（导入材料索引）

> **用途**：记录外部导入知识材料的来源、版本、许可说明与文件索引，便于溯源与增量更新。  
> **公开发布**：本文件仅登记**可公开**的示例索引；用户本地原始 PDF/题库请放在 `source_materials/`（默认不提交二进制）。

---

## 已登记

| 登记日 | 文件 | source_class | 加工产出 | 状态 |
|--------|------|--------------|----------|------|
| 2026-01-15 | [`examples/questions/mock_pmp_set_example_index.md`](../../examples/questions/mock_pmp_set_example_index.md) | `question_bank` + `mixed` | 同上（目录索引） | **Example Dataset**；18 条虚构目录；不批量入库 Mistake |

### 元数据（Mock PMP Set — Example Dataset）

| 字段 | 值 |
|------|-----|
| 标题 | Mock PMP Set — Wrong Question Bank Index（示例） |
| `source_label` | Mock PMP Set — Example Dataset |
| `source_path` | `null`（无仓库内原始 PDF；仅示例 Markdown 索引） |
| `exam_set` | Mock Set A |
| 条目数 | 18（Part 1–3，教学示例） |
| 信任度 | N/A（虚构示例，非真题库） |
| Agent 引用 | 经本索引与 [`examples/questions/mock_pmp_set_example_index.md`](../../examples/questions/mock_pmp_set_example_index.md)；**不**引用用户本地 `source_materials/` 原文 |

---

## 待导入（占位）

| 说明 | 状态 |
|------|------|
| 用户自有教材 / 题库 PDF → 放入 `source_materials/` 后走 `workflows/material_processing.md` | 未在公开仓登记具体文件名 |

---

## 用户本地导入（不写入本索引）

个人学习资料、培训机构题库、本地路径等**不得**写入公开仓库的 `imported_materials.md`。加工完成后仅在本地 `memory/data/` 或通过 Question Coach 单题录入沉淀。
