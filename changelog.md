# Changelog

本仓库遵循 [Semantic Versioning](https://semver.org/) 思路；Skill 版本与 Memory 契约版本（如 `mvp-1.2`）可并存，以 Git 标签 / 本文件为准。

---

## [0.1.0] — 2026-08-04

### 首个可发布 Skill 版本（MVP）

#### 核心数据流

- **三分流**：Question History / Mistake Memory / Bookmark Memory 职责分离  
- **答错自动入库**：`user_answer ≠ correct_answer` → History + Mistake，无需用户确认保存  
- **答对**：仅 History，不写 Mistake  
- **收藏**：仅 Bookmark，不参与错因与错误统计  

#### Question Coach

- `workflows/question_capture.md`：单题采集（最少输入 L0–L3）→ 再进入 `question_analysis.md`  
- `inputs/question_capture_input.md`：采集输入约定  
- `workflows/question_analysis.md` 固定输出与 `DATA_HANDOFF`  
- `modules/question_coach/output_contract.md`：`data_routing`（`write_history` / `write_mistake` / `write_bookmark`）  
- 错因字段对外统一：`error_type`、`error_reason`、`exam_domain`  

#### Mistake Coach

- P0 真实错题判定（`decision_rules.md`）  
- `memory_record_contract.md`：Mistake 写入九字段核心集  
- 自动更新 `weak_points.json`、`learning_state.json`  

#### Review Coach

- `workflows/review_retrospective.md`：复盘**现场重算** Mistake，禁止读快照作输入  
- 错因分析主要来自 Mistake；History 用于趋势、重复作答、争议题  
- `review_retrospective.json`：`record_kind=review_snapshot`、`source_revisions`、`snapshot_status`  

#### Study Planner

- `planning_rules.md`：优先 Mistake + WeakPoint；过期快照不融合  

#### Schema & Memory

- `database/schema_overview.md` §1.1 字段规范（canonical + 历史只读别名）  
- **Skill 与个人运行数据隔离**：公开仓提交 Skill / 工作流 / Schema；`memory/data/*.json` 运行时文件由 `.gitignore` 排除，仅本机积累  
- **公开示例**：`examples/`（Markdown 输出样例）、`memory/data/examples/`（JSON 结构样例，Example Dataset）、`modules/**/examples/` — **不含**作者个人学习记录  

#### 文档与发布

- 根目录 `README.md` 发布说明  
- [`memory/INITIALIZATION.md`](memory/INITIALIZATION.md)：新用户 Memory 从零启动（懒创建 / 空壳 JSON）  
- `examples/`：错题分析、答对、复盘报告样例（已脱敏，可公开）  
- `knowledge/source/imported_materials.md`：仅登记可公开的示例索引（Mock PMP Set — Example Dataset）  
- Release Check：History/Mistake/Bookmark 隔离、闭环、快照一致性 **PASS**  

#### 已知限制（非阻塞）

- 无独立 `memory/data/questions.json` 题库文件（Question 实体仅在 Schema 层）  
- 历史 `mistake_memory.json` 条目不迁移，可能仍含只读别名 `mistake_type`  
- 无自动化 CI / 单元测试；依赖 Agent 按文档执行  

---

## [Unreleased]

---

## [0.1.1] — 2026-08-06

### Answer Validation（最小升级）

- **AEL**：`platform_answer` / `coach_answer` / `adjudication_answer` / `answer_confidence` / `answer_status` / `answer_disputed`（`answer_evaluation` 对象）  
- **判题**：Mistake 与 History `result` 以 **`adjudication_answer`** 为准；`correct_answer` 在 handoff 中与 adjudication 同义  
- **争议**：`answer_disputed=true` 且用户答案 = Coach（高/中置信）→ **不写 Mistake**；争议仅存 History，复盘不计错因 share  
- **文档**：`decision_rules.md` §0、`question_analysis.md` §5.5、`question_capture.md`、`database/question_schema.md` 等  
- **兼容**：无 `answer_evaluation` 的 v0.1.0 记录按 `legacy` 读取；无新 Memory 文件、无新模块  

---
