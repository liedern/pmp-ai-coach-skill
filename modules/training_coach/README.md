# Training Coach（培训教练模块）

> **所属产品**：PMP AI Coach（单一主 Skill）  
> **模块类型**：专业能力模块 | **非独立 Skill**

---

## 模块目标

负责 **PMP 知识点教学、概念辨析、资料加工引导与备考认知建设**，在用户「还没做题或想系统学」时提供结构化讲解。

- 按 `skill.md` Teaching Mode 六段式输出概念讲解
- 检索 `knowledge/` 各领域知识作答
- 触发或配合 `material_processing` 将原始资料转化为结构化知识
- 支持模拟考试训练引导（未来与 `mock_exam` 衔接）

**不负责**：单题对错判断与归档（→ Question Coach）、错题记忆（→ Mistake Coach）、日计划排期（→ Study Planner）。

---

## 用户触发场景

| 场景 | 典型用户说法 |
|------|--------------|
| 概念答疑 | 「什么是 Validate Scope？」「Risk 和 Issue 区别？」 |
| 过程/ITTO 讲解 | 「讲讲实施整体变更控制」 |
| 易混对比 | 「Validate Scope 和 Control Quality 怎么分？」 |
| 敏捷角色 | 「PO 和 SM 分别做什么？」 |
| 上传学习资料 | 「帮我整理这份 PDF 讲义」 |
| 口诀/规则学习 | 「PMP 决策优先级是什么？」 |
| 模拟考试（未来） | 「开始一套模考」 |

---

## 输入

| 输入类型 | 来源 | 说明 |
|----------|------|------|
| 概念/问题 | `inputs/chat_input.md` | `teach` 意图 |
| 学习资料 | `inputs/document_input.md` | PDF/PPT/Word/笔记 |
| 知识检索 query | 用户问题关键词 | 映射 `knowledge/` 路径 |
| 用户薄弱点 | `memory/weak_points.md` | 个性化优先讲弱项 |
| 用户偏好 | `memory/user_profile.md` | 讲解深度、双语术语 |

---

## 输出

| 产出 | 格式 | 说明 |
|------|------|------|
| 结构化讲解 | Markdown 六段式 | 定义 → 考试意义 → 关键词 → 场景 → 陷阱 → 记忆法 |
| 易混对比表 | 表格或对照 | 引用 `confusing_terms.md` |
| 资料加工报告 | `Material Processing Report` | 经 `material_processing` 流水线 |
| 结构化知识 | `knowledge/` 写入 | 加工后的知识点/术语 |
| 可选进度 | `learning_progress` | 标记今日学习内容 |

---

## 数据依赖

### 工作流（Workflows）

| 文件 | 关系 |
|------|------|
| `workflows/material_processing.md` | 资料加工主流水线 |
| `workflows/mock_exam.md` | 模考训练（待完善，P1） |

### 知识库（Knowledge）

| 路径 | 用途 |
|------|------|
| `knowledge/pmbok/*` | 十大知识领域（待填充） |
| `knowledge/agile/*` | 敏捷/Scrum（待填充） |
| `knowledge/exam/*` | 考纲、题型、陷阱 |
| `knowledge/language/*` | 术语、易混、同义词 |
| `knowledge/decision_framework/*` | 决策规则与口诀 |
| `knowledge/terminology/pmp_glossary.md` | 术语总表 |

### 数据模型（Database）

| 文件 | 关系 |
|------|------|
| — | 本模块以知识库为主，少直接写 DB |

### 记忆（Memory）

| 文件 | 读写 |
|------|------|
| `memory/weak_points.md` | 读（优先讲薄弱项） |
| `memory/user_profile.md` | 读 |
| `memory/learning_progress.md` | 写（可选） |

### 原始资料

| 路径 | 关系 |
|------|------|
| `source_materials/` | 只读原料，加工后写入 `knowledge/` |

### 主 Skill

| 文件 | 关系 |
|------|------|
| `skill.md` §6 | Teaching Mode 输出结构 |

---

## 与 Material Processing 的关系

```
用户上传 PDF/PPT
      │
      ▼
Training Coach（识别为资料加工意图）
      │
      ▼
workflows/material_processing.md
      │
      ▼
knowledge/ + imported_materials 索引
```

Agent **不得**直接引用 `source_materials/` 原文作答，须引用加工后的 `knowledge/`。

---

## 模块边界

```
用户概念问题 / 上传资料
        │
        ▼
Training Coach ──讲解──→ 用户
        │
        ├──检索──→ knowledge/*
        └──加工──→ material_processing → knowledge/*
```
