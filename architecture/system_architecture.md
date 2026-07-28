# System Architecture

> PMP AI Coach 整体系统架构说明。描述各层职责、边界与依赖关系。
>
> **版本**：架构增强 v1 | **原则**：平台无关、文档驱动、Agent 可调用

---

## 1. 架构总览

```
┌─────────────────────────────────────────────────────────────────┐
│                        Input Layer                               │
│   question / document / image / chat                             │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────┐
│                        Agent Layer                               │
│   skill.md — 角色、推理框架、输出风格、约束                        │
└────────────────────────────┬────────────────────────────────────┘
                             │
        ┌────────────────────┼────────────────────┐
        ▼                    ▼                    ▼
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│ Workflow Layer│   │Knowledge Layer│   │ Memory Layer  │
│ 业务流程编排   │   │ 领域知识与规则 │   │ 个人学习记忆   │
└───────┬───────┘   └───────┬───────┘   └───────┬───────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                            ▼
                  ┌─────────────────┐
                  │ Database Layer  │
                  │ 持久化数据模型   │
                  └─────────────────┘
```

---

## 2. 各层说明

### 2.1 Input Layer（输入层）

| 项目 | 说明 |
|------|------|
| **路径** | `inputs/` |
| **作用** | 定义不同输入媒介的接收格式、预处理步骤与路由目标 Workflow |
| **输入类型** | 题目文字、PDF/Word 文档、图片截图、自由对话 |
| **输出** | 规范化 `input_payload`（类型、原文、元数据、置信度） |

**关系**：所有用户交互经 Input Layer 标准化后，交给 Agent Layer 选择 Workflow。

---

### 2.2 Agent Layer（智能体层）

| 项目 | 说明 |
|------|------|
| **路径** | `skill.md`（根目录） |
| **作用** | 定义 PMP AI Coach 的角色、使命、五步推理框架、输出风格与硬性约束 |
| **职责** | 意图识别、Workflow 调度、Knowledge 检索、Memory 读写、结构化输出 |
| **不承载** | 具体业务流程细节（属 Workflow）、领域知识全文（属 Knowledge） |

**关系**：Agent 是编排中枢，不替代各层文档，而是按架构引用它们。

---

### 2.3 Workflow Layer（工作流层）

| 项目 | 说明 |
|------|------|
| **路径** | `workflows/` |
| **作用** | 定义可复用的端到端业务流程与输出契约 |
| **核心工作流** | `question_analysis`、`material_processing`、`study_plan`、`mock_exam`、`mistake_classification` |
| **特点** | 步骤化、可审计、含 DATA_HANDOFF / 固定输出模板 |

**关系**：由 Agent 根据 Input 类型触发；执行中检索 Knowledge，结束后更新 Memory / Database。

---

### 2.4 Knowledge Layer（知识层）

| 项目 | 说明 |
|------|------|
| **路径** | `knowledge/` |
| **作用** | 存放 PMP 考试知识、决策规则、术语体系，供 Agent 推理时检索 |
| **子模块** | |

| 子目录 | 内容 |
|--------|------|
| `exam/` | 考纲、题型、陷阱、关键词 |
| `language/` | 中英术语、易混词、同义词映射 |
| `pmbok/` | 十大知识领域 |
| `agile/` | 敏捷 / Scrum / Hybrid |
| `terminology/` | 术语表（与 language 互补） |
| `decision_framework/` | 决策树、优先级规则、预测型/敏捷逻辑 |
| `source/` | 外部导入材料索引 |

**关系**：只读参考库（加工流水线可写入）；Workflow 与 Agent 按场景加载子集。

---

### 2.5 Memory Layer（记忆层）

| 项目 | 说明 |
|------|------|
| **路径** | `memory/` |
| **作用** | 用户个人学习记忆的轻量结构与运行时视图 |
| **模块** | `user_profile`、`mistake_memory`、`weak_points`、`learning_progress` |
| **特点** | 跨会话延续、个性化教练、不替代 Database 完整 Schema |

**关系**：Agent 在每次交互后读写 Memory；可与 Database Layer 同步或作为对话内缓存。

---

### 2.6 Database Layer（数据层）

| 项目 | 说明 |
|------|------|
| **路径** | `database/` |
| **作用** | 产品级持久化数据模型设计（Schema 文档，非 ORM 代码） |
| **实体** | Mistake、Question、Learning 等 |
| **特点** | 字段稳定、枚举可扩展、支持未来迁移至真实数据库 |

**关系**：Workflow 的 Data Handoff 映射到 Schema；Memory 是面向 Agent 的简化视图。

---

## 3. 辅助目录

| 目录 | 作用 |
|------|------|
| `architecture/` | 本文件及数据流、Agent 流（架构元文档） |
| `templates/` | 统一输出模板（错题报告、学习计划等） |
| `examples/` | 示例输入输出、题库样例 |

---

## 4. 层间依赖规则

| 规则 | 说明 |
|------|------|
| **单向依赖** | Input → Agent → Workflow → Knowledge / Memory / Database |
| **Knowledge 不依赖 Memory** | 公共知识与用户数据分离 |
| **Workflow 不嵌入大段知识** | 知识引用路径，内容在 Knowledge Layer |
| **Memory 可引用 Database 字段名** | 保持映射一致，不重复定义枚举 |
| **扩展点** | 新输入类型 → `inputs/`；新知识域 → `knowledge/`；新流程 → `workflows/` |

---

## 5. 部署形态（未来）

| 形态 | 各层对应 |
|------|----------|
| Cursor / ChatGPT Skill | Markdown 文档 + skill.md 入口 |
| Web 产品 | Database 实体化 + API 调用 Workflow 契约 |
| RAG | Knowledge Layer 向量化 + Agent 检索 |
| 多用户 | Memory / Database 按 `user_id` 隔离 |
