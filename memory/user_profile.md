# User Profile（用户档案）

> 个人学习记忆系统 — 用户基础信息与学习偏好。
>
> 本文档定义 `user_profile` 的数据结构与字段约定，**不是**可执行代码。实现层可为每用户维护一份 JSON / Markdown 或数据库 User 扩展表。

---

## 1. Purpose

记录 PMP 考生的**静态与半静态**个人信息，供 Agent 在讲解、规划、提醒时做个性化适配。

| 用途 | 说明 |
|------|------|
| 备考节奏 | 根据目标日期倒推学习强度 |
| 个性化教练 | 匹配学习方式与内容深度 |
| 进度对齐 | 区分「已掌握」与「待学习」，避免重复推送 |
| 主动提醒 | 触发复习、模考、弱项突破等提醒 |

### 与其他 Memory 模块的关系

```
user_profile（本文件）
    ├── 指导 → learning_progress（每日学什么）
    ├── 过滤 → mistake_memory（复习优先级）
    └── 输入 → weak_points（薄弱域报告与建议）
```

---

## 2. Core Fields

| 字段名称 | 类型建议 | 是否必须 | 用途 |
|----------|----------|----------|------|
| `user_id` | UUID / STRING | **是** | 用户唯一标识 |
| `exam_target_date` | DATE | 否 | PMP 考试目标日期；未设定为 `null` |
| `current_study_stage` | ENUM / STRING | 推荐 | 当前学习阶段（见 §3） |
| `learning_style` | ENUM / STRING[] | 否 | 学习方式偏好（见 §4） |
| `mastered_content` | JSON / TEXT[] | 否 | 用户或系统判定已掌握的内容标签列表 |
| `learning_preferences` | JSON | 否 | 学习偏好配置（见 §5） |
| `reminders` | JSON[] | 否 | 需要提醒的事项列表（见 §6） |
| `created_at` | TIMESTAMPTZ | **是** | 档案创建时间 |
| `updated_at` | TIMESTAMPTZ | **是** | 最后更新时间 |

---

## 3. PMP 考试目标日期（exam_target_date）

| 字段 | 说明 |
|------|------|
| `exam_target_date` | 计划参加考试日期，格式 `YYYY-MM-DD` |
| `days_remaining` | **计算字段**：目标日 − 今日；无目标日期时不计算 |
| `exam_version` | 可选：目标考纲版本，如 `ECO_2021` |

**Agent 使用规则**：

- 距考试 ≤ 30 天 → 建议强化模考与错题复习
- 无目标日期 → 提示用户设定，但不编造日期

---

## 4. 当前学习阶段（current_study_stage）

| 存储值 | 中文 | 典型特征 |
|--------|------|----------|
| `not_started` | 未开始 | 刚注册，未系统学习 |
| `foundation` | 基础学习 | 通读教材 / 视频，建立框架 |
| `practice` | 刷题强化 | 以做题为主，积累错题 |
| `review` | 冲刺复习 | 错题 + 弱项 + 模考 |
| `pre_exam` | 考前调整 | 轻量复习，调整状态 |
| `passed` | 已通过 | 备考结束；可归档或切换维持模式 |
| `paused` | 暂停 | 用户主动暂停备考 |

---

## 5. 学习方式（learning_style）

可多选，存储为数组：

| 存储值 | 中文 | Agent 适配 |
|--------|------|------------|
| `video` | 视频课 | 推荐章节、时间点 |
| `reading` | 阅读教材 | 推荐 PMBOK / 讲义段落 |
| `practice_heavy` | 题海战术 | 多推题目与解析 |
| `concept_first` | 先概念后做题 | 先讲再练 |
| `flashcard` | 卡片记忆 | 术语、口诀、一句话规则 |
| `discussion` | 对话式 | 多追问、苏格拉底式讲解 |
| `mock_exam` | 模考驱动 | 限时套卷 + 复盘 |

---

## 6. 已掌握内容（mastered_content）

标签化列表，每项建议结构：

```json
{
  "tag": "risk_management",
  "label": "风险管理",
  "eco_domain": "process",
  "mastered_at": "2026-07-15",
  "confidence": "high",
  "source": "user_declared | system_inferred"
}
```

| 规则 | 说明 |
|------|------|
| `source = user_declared` | 用户自述掌握 |
| `source = system_inferred` | 连续做对 / `review_status = mastered` 推断，标【推测】 |
| 与 `weak_points` 互斥 | 同一标签不应同时出现在 mastered 与 weak 高位 |

---

## 7. 学习偏好（learning_preferences）

```json
{
  "daily_study_minutes": 60,
  "preferred_study_time": "evening",
  "language": "zh",
  "bilingual_terms": true,
  "explanation_depth": "exam_focused",
  "include_agile": true,
  "include_pmbok_process": true,
  "notification_enabled": true
}
```

| 子字段 | 说明 |
|--------|------|
| `daily_study_minutes` | 期望每日学习时长（分钟） |
| `preferred_study_time` | `morning` / `afternoon` / `evening` / `flexible` |
| `language` | 主交互语言 |
| `bilingual_terms` | 是否强制保留英文术语 |
| `explanation_depth` | `brief` / `exam_focused` / `detailed` |

---

## 8. 需要提醒的事项（reminders）

```json
{
  "reminder_id": "uuid",
  "type": "review_mistakes | mock_exam | weak_point | custom",
  "title": "复习昨日错题",
  "schedule": "daily | weekly | once | before_exam",
  "scheduled_at": "2026-07-28T20:00:00+08:00",
  "enabled": true,
  "note": "用户自定义说明"
}
```

| type | 触发场景 |
|------|----------|
| `review_mistakes` | 错题复习到期 |
| `mock_exam` | 模考计划 |
| `weak_point` | 薄弱点专项 |
| `custom` | 用户自定义 |

---

## 9. 数据模板（运行时填充）

```yaml
user_id: null
exam_target_date: null
current_study_stage: not_started
learning_style: []
mastered_content: []
learning_preferences: {}
reminders: []
updated_at: null
```

> **说明**：以上为单用户档案模板。多用户场景下每用户独立一份，不混写。
