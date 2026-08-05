---
id: mock_pmp_set_example_index
type: question_bank
source_class: question_bank
source_label: Mock PMP Set — Example Dataset
source_path: null
exam_set: Mock Set A
item_count: 18
processed_at: 2026-01-15
processor: material_processing (example only)
need_review: true
notes: >
  虚构题库目录索引，演示 material_processing 对「混合笔记型题库」的产出形态。
  进入 Mistake Memory 须经 question_analysis 且具备 user_answer ≠ correct_answer。
---

# Mock PMP Set — 错题集目录索引（示例）

| # | 标题 | 形式 | 选项 | 可单题 Coach |
|---|------|------|------|----------------|
| 1 | 法规变化（Regulatory Change） | 完整单选 | A-D | ✅ |
| 2 | 客户要求增加待办事项 | 情景 + 思路 | 无 | ⚠️ 需补选项或错选 |
| 3 | CEO 抱怨第一迭代性能 | 情景 + 思路 | 无 | ⚠️ |
| 4 | 运营团队验证 vs 运营总监验收标准 | 情景 | 无 | ⚠️ |
| 5 | PMO 与项目经理区别 | 概念 | 无 | 📘 知识卡 |
| 6 | 每日站会 vs 状态报告 | 情景 | 无 | ⚠️ |
| 7 | 分布式团队是否全用正式书面沟通 | 情景 | 无 | ⚠️ |
| 8 | 工资报告泄露致团队冲突 | 情景 | 无 | ⚠️ |
| 9 | 固定总价合同 + 内部约束无法完成迭代 | 情景 | 无 | ⚠️ |
| 10 | Assumption Log 假设日志 | 概念 | 无 | 📘 |
| 11 | Audited Resources 审计资源 | 概念 | 无 | 📘 |
| 12 | 质量审计失败 / 新干系人 / 审计资源 | 情景 | 无 | ⚠️ |
| 13 | 关键路径与浮动时间 | 情景/概念 | 无 | ⚠️ |
| 14 | 依赖关系 FS/SS/FF/SF | 概念 | 无 | 📘 |
| 15 | 干系人利益冲突 | 情景 | 无 | ⚠️ |
| 16 | Facilitate Stakeholder Engagement | 概念 | 无 | 📘 |
| 17 | High-Level Report 高层报告 | 部分单选 | A-B 片段 | ⚠️ |
| 18 | 资源分配文档（RAM / Calendar / Plan） | 概念 | 无 | 📘 |

## 模块分区（示例）

1. **Part 1**：变更管理 + 干系人 + 沟通  
2. **Part 2**：敏捷 + 资源 + 质量  
3. **Part 3**：进度 + 干系人补充 + 通用判断（错误模式总结 4.1–4.3）

## Part 3 归纳（教学示例）

- **4.1** 关键词直配工具为常见陷阱 → 先判根因。  
- **4.2** 不熟悉文件区别（变更计划 vs 子管理计划等）。  
- **4.3** 偏好 Facilitate / Collaborate / Engage，慎用 Force。

## 与运行时 Memory 的关系

- 本文件 **不** 自动写入 `memory/data/mistake_memory.json`。  
- 可与 Skill 内其他 **Example Dataset** 题目（如 EVM、变更控制）做专题串联复习。

## 推荐用法（Question Coach）

对 ✅ / ⚠️ 条目：发送 **题干 + 选项 + 你的答案**（或截图），走 `question_capture` → 自动 History / Mistake。

对 📘 条目：可说「按 Mock Set 条目 16 考我概念」——讲解为主，无作答则不生成 Mistake。
