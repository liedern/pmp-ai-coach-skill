# User Schema

> 用户实体数据模型。对应 `memory/user_profile.md`。  
> **消费模块**：全模块读；Study Planner 写阶段与目标；用户自助更新偏好。

---

## 1. Purpose

`User` 是 PMP AI Coach 中所有个人学习数据的**根实体**。Web 产品扩展时，一条 `User` 记录对应一个考生账号。

| 用途 | 说明 |
|------|------|
| 备考目标 | 考试日期、当前阶段驱动 Study Planner 排期 |
| 个性化教练 | 学习风格、偏好深度影响 Training / Question Coach 输出 |
| 权限与隔离 | 多租户下 `user_id` 隔离 Mistake、Progress 等 |
| 提醒与触达 | Web 产品推送复习提醒、模考提醒 |

---

## 2. Core Fields

| 字段名称 | 类型建议 | 是否必须 | 用途 |
|----------|----------|----------|------|
| `user_id` | UUID | **是** | 主键 |
| `display_name` | VARCHAR(128) | 否 | 展示名 |
| `locale` | VARCHAR(16) | 否 | 语言偏好，默认 `zh-CN` |
| `exam_target_date` | DATE | 否 | PMP 考试目标日期 |
| `exam_version` | VARCHAR(32) | 否 | 考纲版本，如 `ECO_2021` |
| `current_study_stage` | ENUM | 推荐 | 学习阶段，见 §3 |
| `learning_style` | JSON / TEXT[] | 否 | 学习风格偏好，见 §4 |
| `mastered_content` | JSON / TEXT[] | 否 | 已掌握内容标签，供 Planner 过滤 |
| `learning_preferences` | JSON | 否 | 讲解深度、术语双语等，见 §5 |
| `daily_study_minutes` | INTEGER | 否 | 每日可用学习时长（分钟），Planner 排期用 |
| `timezone` | VARCHAR(64) | 否 | 时区，如 `Asia/Shanghai` |
| `created_at` | TIMESTAMPTZ | **是** | 注册/建档时间 |
| `updated_at` | TIMESTAMPTZ | **是** | 最后更新 |

---

## 3. current_study_stage 枚举

| 存储值 | 中文 | 说明 |
|--------|------|------|
| `not_started` | 未开始 | 刚建档 |
| `foundation` | 基础学习 | 系统过教材/知识域 |
| `practice` | 刷题强化 | 以做题为主 |
| `review` | 错题复习 | 以错题与弱项为主 |
| `mock_exam` | 模考冲刺 | 全真模拟 + 查漏补缺 |
| `final_sprint` | 考前冲刺 | 距考试 ≤14 天 |

Study Planner 根据阶段调整 `StudyPlanItem` 类型比例（阅读/刷题/复习/模考）。

---

## 4. learning_style 示例

```json
{
  "learning_style": ["visual", "example_driven"],
  "prefers_bilingual_terms": true,
  "explanation_depth": "exam_focused"
}
```

| 值 | 说明 |
|----|------|
| `visual` | 偏好表格、对比图 |
| `example_driven` | 偏好场景举例 |
| `concise` | 偏好简短结论 |
| `deep` | 偏好展开推理链 |

---

## 5. learning_preferences 示例

```json
{
  "explanation_depth": "exam_focused",
  "show_pmbok_reference": false,
  "auto_save_mistakes": false,
  "review_reminder_enabled": true,
  "review_reminder_time": "20:00"
}
```

---

## 6. 与其他实体关系

```
User (1) ──< (N) Mistake
User (1) ──< (N) WeakPoint
User (1) ──< (N) StudyPlan
User (1) ──< (N) ReviewSession
User (1) ──< (N) LearningProgress
```

---

## 7. Memory 映射

| User 字段 | memory/user_profile.md |
|-----------|------------------------|
| `exam_target_date` | `exam_target_date` |
| `current_study_stage` | `current_study_stage` |
| `learning_style` | `learning_style` |
| `mastered_content` | `mastered_content` |
| `learning_preferences` | `learning_preferences` |

---

## 8. 最小 JSON 示例

```json
{
  "user_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
  "display_name": "考生 A",
  "locale": "zh-CN",
  "exam_target_date": "2026-10-15",
  "exam_version": "ECO_2021",
  "current_study_stage": "practice",
  "learning_style": ["example_driven", "concise"],
  "mastered_content": ["项目章程", "WBS"],
  "learning_preferences": {
    "explanation_depth": "exam_focused",
    "prefers_bilingual_terms": true,
    "auto_save_mistakes": false
  },
  "daily_study_minutes": 90,
  "timezone": "Asia/Shanghai",
  "created_at": "2026-07-01T10:00:00+08:00",
  "updated_at": "2026-07-29T17:00:00+08:00"
}
```
