# PMP AI Coach — 项目长期维护状态

> **用途**：重新打开仓库时快速恢复上下文。  
> **更新约定**：每次发布版本或重大架构决策后更新本文件（与 `changelog.md` 同步）。  
> **最后更新**：2026-08-06

---

## 1. 当前版本

| 项 | 值 |
|----|-----|
| **Skill 发布版本（文档目标）** | **v0.1.1** |
| **Memory 契约** | `mvp-1.3`（Answer Validation） |
| **AEL 版本** | `ael_version` = `0.1.1` |
| **入口** | [`skill.md`](../skill.md) / `SKILL.md`（同内容） |
| **远程仓库** | https://github.com/ryy196/pmp-ai-coach-skill.git |
| **默认发布分支** | `cursor/init-pmp-ai-coach-skill` |

**说明**：当前 **线上发布线** 为 **v0.1.x**（v0.1.0 MVP → **v0.1.1** Answer Validation）。尚未发布 semver **v1.0.0**。Git 记录见 §9；后续发布按 §11 执行。

---

## 2. v0.1.0 已完成功能（首个可发布 MVP）

### 2.1 核心数据流

- **三分流（P0）**：Question History / Mistake Memory / Bookmark Memory 严格分离  
- **答错自动入库**：无需用户说「保存错题」  
- **答对**：仅 History  
- **收藏**：仅 Bookmark，不参与错因与错误统计  
- **无** `questions.json` 题库文件（Question 为逻辑实体 + Schema）

### 2.2 Coach 与工作流

| 能力 | 关键路径 |
|------|----------|
| Question Coach | `workflows/question_capture.md` → `question_analysis.md` |
| Mistake Coach | `workflows/mistake_classification.md`、`decision_rules.md`（v0.1.0 P0） |
| Review Coach | `workflows/review_retrospective.md`；错因重算自 Mistake |
| Study Planner | `workflows/study_plan.md`、`modules/study_planner/` |
| Training Coach | 无完整四选一题目时的概念/资料 |

### 2.3 Schema & Memory

- `database/schema_overview.md`：canonical 字段 `error_type` / `exam_domain` / `error_reason`  
- **Skill 与个人隐私隔离**：`memory/data/*.json` 运行时 `.gitignore`  
- 公开示例：`examples/`、`memory/data/examples/`、`modules/**/examples/`（Example Dataset，已脱敏）  
- [`memory/INITIALIZATION.md`](../memory/INITIALIZATION.md)：懒创建 / 空壳 JSON  

### 2.4 发布工程（v0.1.0）

- 发布提交：`46f5922` — `release: PMP AI Coach Skill v0.1.0`  
- 标签：`v0.1.0`  
- 运行时验证：隔离 worktree 下 Case 1–4 数据流模拟 **PASS**（见历史会话 Runtime Test Report）

---

## 3. v0.1.1 Answer Validation 升级

**目标**：题库标答不一致时避免盲目判错；Coach 基于 PMI 考试逻辑独立分析。

### 3.1 Answer Evaluation Layer（AEL，文档层）

- 对象：`answer_evaluation`（Handoff + History 快照）  
- 字段：`platform_answer`、`coach_answer`、`coach_answer_reason`、`answer_confidence`、`answer_status`、`answer_disputed`、`adjudication_answer`、`ael_version`  
- **判题基准**：`adjudication_answer`（默认 Coach 高/中置信；否则降级 platform）  
- Handoff 中 `correct_answer` **与 adjudication 同义**（v0.1.1 新写入）

### 3.2 分流变化（相对 v0.1.0）

| 场景 | Mistake |
|------|---------|
| 用户 ≠ adjudication | ✅（满足错因等条件） |
| 争议：用户 = Coach ≠ 平台 | ❌（History `answer_disputed`） |
| `uncertain` / 无 adjudication | ❌ |

- **无新 Memory 文件**、无新 Agent 模块  
- 争议 **不** 写入 `mistake_memory.json` 的 `answer_disputed`（仍仅 History）

### 3.3 主要改动文件（v0.1.1）

- `modules/mistake_coach/decision_rules.md`（§0 AEL + §0.2 争议门禁）  
- `workflows/question_analysis.md`（§5.5）、`question_capture.md`  
- `database/question_schema.md`、`schema_overview.md`、`mistake_schema.md`  
- `skill.md`、`README.md`、`changelog.md`、`version.md` 等  

### 3.4 兼容性

- 无 `answer_evaluation` 的旧记录：`answer_status=legacy`，`adjudication_answer` ← 原 `correct_answer`  
- 不迁移历史 JSON 键名  

---

## 4. 当前架构（简图）

```
用户输入（文字 / 截图 / 「复盘」/ 「今天学什么」）
        │
        ▼
   skill.md（角色 + 静默路由）
        │
        ├── Question Coach ──► question_capture → question_analysis
        │         │                    │
        │         │              [v0.1.1] AEL → adjudication_answer
        │         ├─► question_history.json（每次有作答）
        │         ├─► mistake_memory.json（仅确认错题）
        │         └─► bookmark_memory.json（仅收藏）
        │
        ├── Mistake Coach ──► weak_points.json, learning_state.json
        ├── Review Coach ──► 重算 Mistake；History 趋势/争议；可选 review_retrospective 快照
        ├── Study Planner ──► 计划任务（优先 Mistake + WeakPoint）
        └── Training Coach ──► 概念 / material_processing（无完整做题）

知识：knowledge/   契约：database/   模块契约：modules/
```

**运行时**：Agent 读 Markdown 工作流 + 读写本机 `memory/data/*.json`；**非**独立可执行服务。

---

## 5. 核心设计原则（长期不变）

1. **用户无感**：对外只有一个「PMP AI Coach」，不暴露内部模块名。  
2. **三分流 P0**：Mistake ≠ Bookmark ≠ History；收藏不参与错因统计。  
3. **答错自动沉淀**：禁止「是否保存错题」式交互（`force_skip` 除外）。  
4. **事实与推测分离**：【事实】/【推测】/【待确认】。  
5. **复盘源数据**：错因 **仅** 来自 `mistake_memory.json`；`review_retrospective.json` 仅为写后快照，不作复盘输入。  
6. **Skill 公开、Memory 私有**：克隆仓库不带作者个人做题数据。  
7. **v0.1.1+ 答案可信**：题库标答 = `platform_answer`（来源事实）；判题以 `adjudication_answer`（Coach 考试逻辑）为准；争议不污染 Mistake。  
8. **平台无关 Core**：能力在 `skill.md` + `workflows/` + `database/`；各 Agent 平台通过各自 Skill 入口适配（见未完成项）。

---

## 6. 已完成事项（累计）

- [x] MVP 架构与多 Coach 模块契约  
- [x] Question Capture + Analysis 工作流与 DATA_HANDOFF  
- [x] Mistake / Review / Study Planner 工作流与聚合规则  
- [x] Memory 初始化与 `.gitignore` 隔离  
- [x] 公开 examples 脱敏与 `imported_materials` 清理  
- [x] v0.1.0 Git 发布（分支 + `v0.1.0` tag）与推送  
- [x] v0.1.0 运行时验证（模拟数据流）  
- [x] v0.1.1 Answer Validation 文档与规则 **已发布**（见 §9、§11）  
- [x] 本维护状态文档 `docs/PROJECT_STATE.md`  

---

## 7. 未完成事项

### 7.1 发布与仓库

- [x] **v0.1.1** Git 提交 + 标签 + push（见 §9）  
- [ ] 工作区 **未提交** 文档（`architecture/` 等）— 后续小版本或 docs 提交  
- [ ] 根目录 `.DS_Store` 曾进远程历史；建议 `.gitignore` 强化并停止跟踪（若尚未做）  
- [ ] `scripts/` 未纳入发布范围；是否公开或删除需决策  

### 7.2 产品与工程

- [ ] **无 CI / 无自动化契约测试**（依赖 Agent 按文档执行）  
- [ ] **多平台 Skill 绑定层**（`.cursor/skills`、`.codebuddy/skills` 等薄入口 README）  
- [ ] **国内镜像**（Gitee / Release zip）与「WorkBuddy / CodeBuddy」使用说明  
- [ ] `architecture/*.md` 与 v0.1.1 AEL 全文对齐（当前可能滞后）  
- [ ] `modules/question_coach/output_contract.md` 与 `answer_evaluation` handoff 显式对齐  
- [ ] 可选：`pmp_capability_profile.json` 与 Study Planner 深度融合（非 MVP 必需）  

### 7.3 已知限制（延续）

- 截图 OCR 质量依赖客户端/模型，非仓库可保证  
- 无 Web 产品 / 无真实 DB，仅 JSON Memory  
- 历史 Mistake 条可能含只读别名 `mistake_type` 等，不迁移  

---

## 8. 下一版本规划（建议）

| 版本 | 主题 | 可能内容 |
|------|------|----------|
| **v0.1.1** | Answer Validation 发布 | **已完成** |
| **v0.1.2** | 契约对齐 | `output_contract`、architecture 图更新 AEL；示例 JSON 含 `answer_evaluation` |
| **v0.2.0** | 平台与分发 | `docs/PLATFORMS.md`、CodeBuddy/Cursor 薄 SKILL 壳；国内镜像说明 |
| **v0.3.0** | 质量 | 轻量 schema 校验脚本（非业务逻辑）、golden handoff 样例 |
| **更远** | 产品化 | Web API 映射 `database/`；仍保持 Skill Core 中立 |

**非目标（除非产品方向变更）**：多平台爬取、全球标准答案库、第四类 Memory 文件（争议继续用 History）。

---

## 9. Git Release 记录

| 标签 | 提交 | 日期 | 说明 |
|------|------|------|------|
| **v0.1.0** | `46f5922db25825461301ed4dfa21e3b3925e09a7` | 2026-08-04 | 首个发布：`release: PMP AI Coach Skill v0.1.0` |
| **v0.1.1** | *见下方本次 push 后* | 2026-08-06 | Answer Validation + `docs/PROJECT_STATE.md` |

**分支**：`cursor/init-pmp-ai-coach-skill`（跟踪 `origin`）

**克隆与使用**：

```bash
git clone https://github.com/ryy196/pmp-ai-coach-skill.git
cd pmp-ai-coach-skill
# 默认分支即发布分支；严格对齐标签：git checkout v0.1.1
```

在 Cursor / 其他 Agent 中加载 `skill.md`，运行时数据写入本机 `memory/data/`。

---

## 10. 快速恢复清单（打开项目后 2 分钟）

1. 读 [`skill.md`](../skill.md) 路由与 § 答案可信原则。  
2. 读 [`changelog.md`](../changelog.md) 最近版本。  
3. 做题链路：`workflows/question_capture.md` → `question_analysis.md` → `decision_rules.md`。  
4. 复盘：`workflows/review_retrospective.md` + `aggregation_rules.md`。  
5. 数据契约：`database/schema_overview.md`。  
6. 本文件 §1、§7、§9、§11 看发布与欠账。  

---

## 11. 发布流程（v0.1.1 起，后续版本照此执行）

**原则**：只发 Skill 文档与契约；**不改业务逻辑代码**；**不提交个人 Memory**。

### 11.1 纳入 `git add` 的典型范围

- Answer Validation / 当次版本相关：`workflows/`、`modules/`、`database/`、`skill.md`、`inputs/`（按需）  
- 发布说明：`README.md`、`changelog.md`、`version.md`  
- 维护文档：`docs/PROJECT_STATE.md`  
- Memory 说明：`memory/INITIALIZATION.md`（及确认为 **公开模板** 的 `memory/user_profile.json`）  
- 公开示例：`examples/`（脱敏）

目录级 add 时优先 **显式路径**，避免 `git add -A`。

### 11.2 禁止纳入提交

| 禁止 | 原因 |
|------|------|
| `.obsidian/` | 本地 IDE |
| `scripts/` | 未纳入发布范围（除非未来单独决策） |
| `*.DS_Store` | 系统垃圾 |
| `memory/data/*.json`（根目录运行时） | 个人学习数据（`.gitignore`） |
| `memory/user_profile.json` | 仅当含个人备考信息时不提交；空模板可提交 |

### 11.3 发布前检查

```bash
git status
git diff --cached --name-only
# 确认无 .obsidian、scripts、.DS_Store、memory/data 运行时 JSON
```

### 11.4 发布命令（顺序）

```bash
git commit -m "release: PMP AI Coach Skill v0.1.x"
git tag -a v0.1.x -m "PMP AI Coach Skill v0.1.x"
git push -u origin HEAD
git push origin v0.1.x
```

发布后：更新本文件 §9 提交 hash、`changelog.md` 日期（若需）。

---

## 相关链接

- [README](../README.md)  
- [CHANGELOG](../changelog.md)  
- [Version](../version.md)  
- [Memory 初始化](../memory/INITIALIZATION.md)
