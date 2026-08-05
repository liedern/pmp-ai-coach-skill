# Memory 初始化指南

> **受众**：首次安装 PMP AI Coach Skill 的用户  
> **版本**：与 Memory 契约 `mvp-1.2` 对齐  
> **原则**：Skill 提供通用能力；`memory/data/*.json` 为**你个人**的学习状态，默认不随公开仓库分发（见 `.gitignore`）。

---

## 1. 第一次安装流程

### 1.1 安装 Skill

1. 克隆或导入本仓库，在 Agent 平台（如 Cursor）中加载 [`skill.md`](../skill.md) 或 `SKILL.md` 作为 Skill / 规则上下文。  
2. 确认仓库根目录存在 `memory/data/`（公开克隆后通常仅有 `memory/data/.gitkeep`）。  
3. 可选：阅读 [`memory/user_profile.json`](user_profile.json)（仓库内为**空模板**），按需填写考试目标日期等；不影响首次做题。

### 1.2 初始化 Memory（两种方式）

**方式 A — 推荐：懒创建（零配置）**

1. 直接向 Coach 发送一道题（粘贴题干 + 选项 +「我选 X」或上传截图）。  
2. Agent 按 [`workflows/question_capture.md`](../workflows/question_capture.md) → [`workflows/question_analysis.md`](../workflows/question_analysis.md) 执行。  
3. 首次写入时**自动创建**所需的 `memory/data/*.json` 文件（从空列表或默认壳开始追加）。  
4. 答错 → 自动写入 `mistake_memory.json` 并聚合 `weak_points.json`；无需说「保存错题」。

**方式 B — 手动：预置空文件**

若你希望目录中提前存在全部 JSON（便于备份或外部工具读取），可将下文 [§3 空状态示例](#3-空状态示例) 复制到 `memory/data/` 对应文件名。  
**不要**将 `memory/data/examples/` 下的文件直接复制为运行时数据（其中含虚构样例记录，见 §4）。

### 1.3 验证是否初始化成功

| 检查 | 预期 |
|------|------|
| 做完 1 道题（有作答） | `question_history.json` 出现 1 条 `history` |
| 该题答错 | `mistake_memory.json` 出现 1 条 `mistakes` |
| 说「复盘」 | 可读 Mistake；可选生成 `review_retrospective.json` |
| 说「收藏这题」 | `bookmark_memory.json` 增加 1 条（与对错无关） |

字段与写入规则详见 [`memory/README.md`](README.md)、[`database/schema_overview.md`](../database/schema_overview.md)。

---

## 2. 默认 Memory 结构

运行时数据目录：**`memory/data/`**（本地生成，默认**不提交** Git）。

| 文件 | 实体 | 何时需要 | 谁写入 |
|------|------|----------|--------|
| `question_history.json` | QuestionHistory | 第一次**有作答**的做题 | Question Coach 自动 |
| `mistake_memory.json` | Mistake | 第一次**答错** | Mistake Coach 自动 |
| `bookmark_memory.json` | Bookmark | 第一次**主动收藏** | Question Coach 分流 |
| `weak_points.json` | WeakPoint | 第一次答错并聚合后 | Mistake 工作流聚合 |
| `learning_state.json` | 学习事件 / 统计 | 做题、复盘等事件 | 各 Coach 追加 |
| `review_retrospective.json` | 复盘快照 | 第一次用户说「复盘」后（可选缓存） | Review Coach 写后快照 |

**说明**

- 无独立 `questions.json` 题库文件；`question_id` 在 History / Mistake 中关联逻辑题目（见 [`database/question_schema.md`](../database/question_schema.md)）。  
- `pmp_capability_profile.json` 等为**可选**扩展画像，非启动必需；有复盘与错题积累后可由 Coach 生成。  
- 单用户 MVP：`user_id` 固定为 `default_user`（字段预留多用户）。

**相关说明文档（非运行时 JSON）**

| 路径 | 用途 |
|------|------|
| [`memory/README.md`](README.md) | Memory 总览与三分流 |
| [`memory/user_profile.md`](user_profile.md) | 用户档案字段说明 |
| [`memory/mistake_memory.md`](mistake_memory.md) | Mistake 说明 |
| [`database/`](../database/) | 正式 Schema 契约 |

---

## 3. 空状态示例

首次使用前（或手动预置时），各文件应为**空集合**或**无业务记录**。以下为最小合法壳（`mvp-1.2`）；Agent 懒创建时可等价于这些结构。

### question_history.json

- **语义**：做题轨迹；对错都记。  
- **空状态**：`history` 为 `[]`。

```json
{
  "version": "mvp-1.2",
  "user_id": "default_user",
  "history": [],
  "updated_at": null
}
```

### mistake_memory.json

- **语义**：仅真实错题（`user_answer` ≠ `correct_answer`）。  
- **空状态**：`mistakes` 为 `[]`。

```json
{
  "version": "mvp-1.2",
  "user_id": "default_user",
  "mistakes": [],
  "updated_at": null
}
```

### bookmark_memory.json

- **语义**：用户主动收藏；**不参与**错因统计。  
- **空状态**：`bookmarks` 为 `[]`。

```json
{
  "version": "mvp-1.2",
  "user_id": "default_user",
  "bookmarks": [],
  "updated_at": null
}
```

### weak_points.json

- **语义**：由 Mistake **聚合**的薄弱点；无错题时通常不存在条目。  
- **空状态**：`weak_points` 为 `[]`；`last_aggregated_at` 为 `null`。

```json
{
  "version": "mvp-1.2",
  "user_id": "default_user",
  "weak_points": [],
  "last_aggregated_at": null
}
```

### learning_state.json

- **语义**：会话事件与累计统计。  
- **空状态**：无 session、统计为 0。

```json
{
  "version": "mvp-1.2",
  "user_id": "default_user",
  "current_study_stage": "not_started",
  "error_patterns": {
    "by_error_type": {},
    "by_knowledge_point": {},
    "top_patterns": []
  },
  "sessions": [],
  "stats": {
    "total_questions_analyzed": 0,
    "total_mistakes_saved": 0,
    "total_bookmarked": 0,
    "total_wrong": 0,
    "total_disputed": 0,
    "total_reviews": 0,
    "accuracy_rate": null
  },
  "updated_at": null
}
```

### review_retrospective.json

- **语义**：复盘**写后快照**；复盘时必须从 Mistake 等**源数据重算**，不得读本文件作输入。  
- **空状态**：首次安装可**不创建**该文件；若预置，可无 `last_retrospective` 或 `snapshot_status` 表示尚未复盘。

```json
{
  "version": "mvp-1.2",
  "user_id": "default_user",
  "record_kind": "review_snapshot",
  "snapshot_status": "absent",
  "generated_at": null,
  "source_revisions": {},
  "notes": "尚未执行复盘；用户说「复盘」后由 Review Coach 写入。",
  "last_retrospective": null,
  "snapshot_history": []
}
```

### 首次使用时的行为摘要

| 组件 | 首次使用时 |
|------|------------|
| History | 空 → 第一道题后追加 |
| Mistake | 空 → 仅答错后追加 |
| Bookmark | 空 → 仅收藏后追加 |
| WeakPoint | 空 → 有错题聚合后生成 |
| learning_state | 空统计 → 随事件增长 |
| review_retrospective | 可无文件或 absent → 首次「复盘」后写入 |

---

## 4. examples 使用方式

目录：**[`memory/data/examples/`](data/examples/)**

| 要点 | 说明 |
|------|------|
| **性质** | 脱敏的**结构示例**，用于对照字段名、嵌套关系与版本号 |
| **不是** | 你的真实错题、历史或薄弱点 |
| **勿直接复制** | 不要将 `examples/*.json` 覆盖到 `memory/data/*.json`，否则会带入虚构 `mistake_id` / 题目 |
| **正确用法** | 阅读样例 → 对照 [`database/`](../database/) Schema → 自行维护空壳（§3）或交给 Agent 懒创建 |

示例文件列表：

- `mistake_memory.json` — 单条虚构冲突题错题  
- `weak_points.json` — 与上对应的薄弱点  
- `learning_state.json` — 单次 session 示意  
- `review_retrospective.json` — 极简快照示意  

用户可见输出样例另见仓库根目录 [`examples/`](../examples/)（Markdown，非 Memory 运行时）。

---

## 5. 个人数据隔离说明

```
┌─────────────────────────────────────┐
│  PMP AI Coach Skill（公开仓库）      │
│  skill.md · workflows · knowledge   │
│  database schema · 通用推理与路由    │
└─────────────────────────────────────┘
                  │
                  │ 安装 / 加载
                  ▼
┌─────────────────────────────────────┐
│  你的本地 memory/data/*.json         │
│  错题 · 历史 · 收藏 · 薄弱点 · 复盘   │
│  （.gitignore，不随 Skill 发布）      │
└─────────────────────────────────────┘
```

| 分层 | 内容 | 是否共享给他人 |
|------|------|----------------|
| **Skill** | 工作流、知识库、Schema、示例文档 | 是（GitHub / Skill 平台） |
| **Memory** | `memory/data/*.json` 运行时 | **否**（仅本机） |
| **user_profile.json** | 仓库内为模板；你本地修改的备考偏好 | 请勿提交含个人轨迹的定制内容 |

克隆本仓库**不会**继承任何作者的错题、做题历史、学习画像或薄弱点。你的 Memory 从空状态或第一道题开始独立积累。

更多发布说明见 [`memory/README.md` §6](README.md#6-公开发布与本地运行时发布工程)。

---

## 相关链接

- [Memory 总览](README.md)  
- [Schema 总览](../database/schema_overview.md)  
- [根目录 README — Memory 初始化](../README.md#memory-初始化)  
- [Question Capture 最少输入](../inputs/question_capture_input.md)
