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

（后续版本在此记录。）
