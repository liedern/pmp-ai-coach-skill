# Learning Progress（学习进度）

> 个人学习记忆系统 — 按时间轴记录学习过程与成效。
>
> 本文档定义 `learning_progress` 的数据结构，用于日记式进度追踪与「下一步计划」衔接。

---

## 1. Purpose

记录用户**每天/每次学习会话**做了什么、效果如何，支撑：

| 用途 | 说明 |
|------|------|
| 学习轨迹 | 备考过程可追溯、可复盘 |
| 节奏监控 | 做题量、正确率是否达标 |
| 掌握度变化 | 与 `user_profile.mastered_content` 联动 |
| 计划闭环 | `下一步计划` 驱动次日任务 |

### 与其他 Memory 模块的关系

```
learning_progress（本文件，时间轴）
    ↑ 汇总自  做题、复习、看课、模考
    ↓ 输出到  user_profile（阶段推进）
    ↓ 引用    weak_points（明日优先弱项）
    ↓ 引用    mistake_memory（今日复习了哪些错题）
```

---

## 2. Core Fields

每条进度记录（通常对应**一个学习日**或**一次学习会话**）：

| 字段名称 | 类型建议 | 是否必须 | 用途 |
|----------|----------|----------|------|
| `study_date` | DATE | **是** | 学习日期，`YYYY-MM-DD` |
| `learning_content` | TEXT / JSON[] | **是** | 学习内容摘要（见 §3） |
| `questions_attempted` | INTEGER | 否 | 做题数量 |
| `accuracy_rate` | DECIMAL / STRING | 否 | 正确率，如 `0.75` 或 `"75%"`；未做题为 `null` |
| `mastery_status` | TEXT / JSON | 推荐 | 掌握情况描述（见 §4） |
| `next_plan` | TEXT / JSON[] | 推荐 | 下一步计划（见 §5） |

### 建议附加字段

| 字段名称 | 用途 |
|----------|------|
| `progress_id` | 记录 ID |
| `user_id` | 所属用户 |
| `session_type` | `daily` / `mock_exam` / `review` / `reading` |
| `study_duration_minutes` | 本次学习时长 |
| `questions_correct` | 做对题数（便于计算正确率） |
| `mistakes_reviewed` | 复习错题数量 |
| `weak_points_touched` | 本次涉及的薄弱域标签 |
| `notes` | 用户自由笔记 |
| `created_at` | 记录创建时间 |

---

## 3. learning_content（学习内容）

支持纯文本或结构化列表：

```json
{
  "learning_content": [
    { "type": "reading", "topic": "风险管理", "ref": "knowledge/pmbok/risk.md" },
    { "type": "practice", "topic": "变更管理模拟题", "count": 20 },
    { "type": "review", "topic": "错题复习", "mistake_ids": ["q_001", "q_002"] },
    { "type": "video", "topic": "敏捷 Scrum 角色", "duration_min": 45 }
  ]
}
```

| type | 说明 |
|------|------|
| `reading` | 教材 / 知识库阅读 |
| `practice` | 刷题 |
| `review` | 错题或弱项复习 |
| `video` | 视频课 |
| `mock_exam` | 整套模考 |
| `concept` | 概念精讲 / Agent 教学 |

---

## 4. mastery_status（掌握情况）

描述**本次学习后**的状态，非全局档案（全局见 `user_profile.mastered_content`）。

```json
{
  "mastery_status": {
    "newly_mastered": ["风险登记册 vs 问题日志"],
    "still_weak": ["冲突管理", "变更 First 题"],
    "confidence": "medium",
    "summary": "风险管理概念清晰；冲突类情境题仍易选升级选项"
  }
}
```

| 子字段 | 说明 |
|--------|------|
| `newly_mastered` | 本次新掌握标签 |
| `still_weak` | 本次仍薄弱标签 |
| `confidence` | `low` / `medium` / `high` |
| `summary` | 一句话复盘 |

---

## 5. next_plan（下一步计划）

可执行、可检验的后续任务：

```json
{
  "next_plan": [
    { "date": "2026-07-29", "action": "复习 conflict_management 薄弱点 5 题", "priority": "high" },
    { "date": "2026-07-29", "action": "阅读 knowledge/pmbok/resource.md 冲突章节", "priority": "medium" },
    { "date": "2026-07-30", "action": "完成模拟卷第3套（60题限时）", "priority": "high" }
  ]
}
```

**Agent 生成规则**：

- 优先引用 `weak_points.suggested_direction`
- 结合 `user_profile.exam_target_date` 倒排强度
- 计划项应具体（数量、主题、时间），避免空泛「继续学习」

---

## 6. 正确率（accuracy_rate）

| 计算 | 说明 |
|------|------|
| `accuracy_rate = questions_correct / questions_attempted` | 当两者均有值时 |
| 仅复习无做题 | `accuracy_rate = null`，不编造 |
| 模考 | 可单独记录 `session_type: mock_exam` |

---

## 7. 单条记录示例

```json
{
  "study_date": "2026-07-28",
  "session_type": "daily",
  "study_duration_minutes": 90,
  "learning_content": [
    { "type": "practice", "topic": "People 领域情境题", "count": 15 },
    { "type": "review", "topic": "错题复习", "mistake_ids": ["q_conflict_001"] }
  ],
  "questions_attempted": 15,
  "questions_correct": 11,
  "accuracy_rate": 0.73,
  "mastery_status": {
    "still_weak": ["冲突管理"],
    "summary": "协作优先于升级仍不稳定"
  },
  "next_plan": [
    { "date": "2026-07-29", "action": "冲突管理专项 10 题", "priority": "high" }
  ]
}
```

---

## 8. Agent 读写规则

| 操作 | 规则 |
|------|------|
| **日终总结** | 用户说「今天学了什么」或会话结束时，可提议写入一条 progress |
| **不编造** | 未做题不填 `accuracy_rate`；未学习内容不虚构 |
| **衔接** | 写入 `next_plan` 后，次日会话优先检查是否执行 |
| **阶段推进** | 连续 N 天达标可建议更新 `user_profile.current_study_stage` |

---

## 9. 数据模板（运行时填充）

```yaml
progress_log: []
# 按 study_date 降序；每条见 §7 结构
```
