# Mistake Memory（错题记忆）

> 个人学习记忆系统 — 用户个人错题与复习状态的轻量记忆层。
>
> 本文档定义 Agent 可读写、可跨会话延续的**错题记忆**字段。持久化实现可对齐 `database/mistake_schema.md`，本文件侧重教练对话与个性化复盘所需的记忆视图。

---

## 1. Purpose

在「题目分析 → 入库」之外，为 AI 教练提供**可引用的个人错题记忆**，支持：

| 用途 | 说明 |
|------|------|
| 跨会话延续 | 「你上次这道题也错了」 |
| 重复错误识别 | `repeated_count` 驱动模式分析 |
| 复习调度 | `review_status` 决定今日复习清单 |
| 改进追踪 | `improvement_note` 记录用户纠偏心得 |

### 与 database/mistake_schema 的关系

| 层级 | 说明 |
|------|------|
| `database/mistake_schema.md` | 产品级持久化 Schema（完整字段、状态机） |
| `memory/mistake_memory.md` | Agent 记忆层：核心字段 + 复习上下文，便于 Markdown/JSON 轻量存储 |

入库时：`question_analysis` Data Handoff → 本结构 → 可选同步至数据库。

---

## 2. Core Fields

| 字段名称 | 类型建议 | 是否必须 | 用途 |
|----------|----------|----------|------|
| `question_id` | UUID / STRING | 推荐 | 题目唯一标识；无主题库时可用内容指纹 `hash` |
| `question_source` | STRING | 否 | 题目来源（题库名、套卷、PMBOK、自编等） |
| `knowledge_point` | STRING / STRING[] | 推荐 | 关联知识点标签，如 `["冲突管理","Collaborate Before Escalate"]` |
| `user_answer` | STRING | 否 | 用户作答选项，如 `"A"`；未提供为 `null` |
| `correct_answer` | STRING | 否 | 正确答案；不确定为 `null` + Need Review |
| `mistake_type` | ENUM / STRING | 否 | 错因分类，枚举见 §3；无用户答案时为 `null` |
| `mistake_reason` | TEXT | 否 | 错因说明：选项差异、思维误区、一句话依据 |
| `repeated_count` | INTEGER | **是** | 同一题（或同一知识点同类题）累计错误次数，默认 `1` |
| `review_status` | ENUM | **是** | 复习状态，枚举见 §4，默认 `new` |
| `improvement_note` | TEXT | 否 | 用户或 Agent 记录的改进笔记、口诀、自查项 |

### 建议附加字段（记忆层扩展）

| 字段名称 | 用途 |
|----------|------|
| `user_id` | 所属用户 |
| `mistake_id` | 与数据库 Mistake 主键对齐 |
| `last_wrong_at` | 最近一次答错时间 |
| `last_reviewed_at` | 最近一次复习时间 |
| `memory_rule` | 一句话记忆规则（来自题目分析） |
| `eco_domain` | People / Process / Business Environment |

---

## 3. mistake_type（错因分类）

与 `mistake_schema` / `question_analysis` 保持一致，**固定枚举**：

| 存储值 | 英文名称 |
|--------|----------|
| `knowledge_gap` | Knowledge Gap |
| `concept_confusion` | Concept Confusion |
| `scenario_judgment_error` | Scenario Judgment Error |
| `process_sequence_error` | Process Sequence Error |
| `role_and_responsibility_error` | Role and Responsibility Error |
| `terminology_problem` | Terminology Problem |
| `carelessness` | Carelessness |
| `insufficient_information` | Insufficient Information |

**规则**：无 `user_answer` 时不得填写 `mistake_type`。

---

## 4. review_status（复习状态）

与 `mistake_schema.md` 生命周期对齐：

| 存储值 | 中文 | 含义 |
|--------|------|------|
| `new` | 新错题 | 刚入错题记忆，未系统复习 |
| `learning` | 学习中 | 已开始针对本题/知识点学习 |
| `reviewing` | 复习中 | 已复习至少一次，未达掌握 |
| `mastered` | 已掌握 | 复习达标或连续做对 |
| `ignored` | 已忽略 | 用户标记不再复习 |

**repeated_count 更新规则**：

- 同一 `question_id` 再次答错 → `repeated_count += 1`，`review_status` 可从 `mastered` 回退至 `reviewing`
- 仅复习未重做 → 不增加 `repeated_count`

---

## 5. 单条记录示例

```json
{
  "question_id": "q_conflict_001",
  "question_source": "某某机构模拟卷第2套 Q15",
  "knowledge_point": ["冲突管理", "Collaborate Before Escalate"],
  "user_answer": "A",
  "correct_answer": "B",
  "mistake_type": "scenario_judgment_error",
  "mistake_reason": "跳过与双方沟通，直接上报发起人",
  "repeated_count": 2,
  "review_status": "reviewing",
  "improvement_note": "团队冲突：先面对面，再升级",
  "last_wrong_at": "2026-07-28T10:00:00+08:00"
}
```

---

## 6. Agent 读写规则

| 操作 | 规则 |
|------|------|
| **写入** | 用户确认保存错题 / 加入错题本后写入；不擅自编造答案与错因 |
| **读取** | 讲解同类题时查询相同 `knowledge_point` 或 `mistake_type` |
| **更新** | 重复做错更新 `repeated_count`；复习后更新 `review_status`、`improvement_note` |
| **引用话术** | 「这道题你错了 {repeated_count} 次，主要原因是 {mistake_type}」 |

---

## 7. 数据模板（运行时填充）

```yaml
mistakes: []
# 每条见 §5 结构
```
