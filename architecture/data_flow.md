# Data Flow

> PMP AI Coach 数据流转定义：从用户输入到知识检索、回答生成与记忆更新的完整路径。
>
> **关联**：`system_architecture.md`、`agent_flow.md`、`inputs/*`

---

## 1. 主数据流（通用）

所有交互遵循以下主干流程：

```
1. 用户输入问题 / 资料 / 对话
        │
        ▼
2. 输入识别（Input Layer）
   - 判定 input_type：question | document | image | chat
   - 预处理：OCR / 解析 / 元数据提取
   - 输出：input_payload
        │
        ▼
3. 调用对应 Workflow（Workflow Layer）
   - Agent 按意图路由
   - 执行步骤化流程
        │
        ▼
4. 检索 Knowledge（Knowledge Layer）
   - decision_framework / exam / language / pmbok / agile
   - 按场景加载，非全库扫描
        │
        ▼
5. 生成回答（Agent Layer）
   - 应用 skill.md 推理框架与输出模板
   - 区分【事实】/【推测】/ Need Review
        │
        ▼
6. 更新 Memory（Memory Layer）
   - user_profile / mistake_memory / weak_points / learning_progress
   - 可选同步 Database Layer
```

---

## 2. 三条业务数据流

### 2.1 知识数据流（Knowledge Data Flow）

**场景**：用户上传 PDF/Word/讲义、或 Material Processing 批量入库。

```
Document / Markdown / Transcript（inputs/document_input）
        │
        ▼
Material Processing Workflow
        │
        ├─ Material Classification → exam | pmbok | terminology | decision_framework | …
        ├─ Information Extraction → concept | term | question fields
        ├─ Knowledge Structuring → Markdown + MATERIAL_HANDOFF JSON
        └─ Quality Validation → Need Review 过滤
        │
        ▼
Knowledge Layer 写入
   knowledge/exam/ | pmbok/ | language/ | decision_framework/ | …
        │
        ▼
knowledge/source/imported_materials.md（索引登记）
```

| 数据形态 | 存储位置 |
|----------|----------|
| 结构化知识点 | `knowledge/pmbok/*.md`、`knowledge/exam/*.md` |
| 术语条目 | `knowledge/language/*.md` |
| 决策规则 | `knowledge/decision_framework/*.md` |
| 题库样例 | `examples/questions/` |

---

### 2.2 错题数据流（Mistake Data Flow）

**场景**：用户提交题目、截图错题、或要求加入错题本。

```
Question Text / Image（inputs/question_input | image_input）
        │
        ▼
[可选] OCR → 题目结构化
        │
        ▼
Question Analysis Workflow
        │
        ├─ Input Extraction → 题干、选项、答案
        ├─ PMP Analysis Framework → 类型、阶段、ECO、错因
        ├─ Answer Explanation
        └─ Data Handoff JSON
        │
        ├──────────────────┬──────────────────┐
        ▼                  ▼                  ▼
Memory Layer         Database Layer      Knowledge（只读引用）
mistake_memory.md    mistake_schema      decision_tree
        │                  │
        ▼                  ▼
weak_points 聚合    持久化 Mistake 记录
（mistake_memory → weak_points.md）
```

| 阶段 | 关键字段 |
|------|----------|
| 分析产出 | `question_id`, `mistake_type`, `knowledge_points`, `explanation` |
| 记忆写入 | `mistake_memory`: repeated_count, review_status, improvement_note |
| 库表映射 | `database/mistake_schema.md` 全字段 |

---

### 2.3 学习数据流（Learning Data Flow）

**场景**：每日学习、刷题统计、学习计划、模考复盘。

```
Chat / 学习汇报（inputs/chat_input）
   或 刷题/模考会话结束
        │
        ▼
Study Plan Workflow | Mock Exam Workflow（未来）
        │
        ├─ 读取 memory/user_profile（目标日期、阶段、偏好）
        ├─ 读取 memory/weak_points（薄弱域）
        └─ 读取 memory/mistake_memory（待复习题）
        │
        ▼
生成：今日任务 / 学习计划 / 模考报告
        │
        ▼
Memory Layer 更新
   learning_progress.md（study_date, accuracy_rate, next_plan）
   user_profile.md（mastered_content, current_study_stage）
   weak_points.md（重新聚合 error_trend）
```

| 输入 | 输出 |
|------|------|
| 今日做题 20 道，正确 15 | `learning_progress` 一条记录 |
| 连续 3 天弱项改善 | `weak_points.error_trend` → falling |
| 考前 14 天 | `study_plan` 提高模考与错题权重 |

---

## 3. 输入类型 → Workflow 路由表

| input_type | 预处理 | 主 Workflow | 次要 Workflow |
|------------|--------|-------------|---------------|
| `question` | 结构化题干选项 | `question_analysis` | — |
| `image` | OCR | `question_analysis` | `material_processing`（非题图） |
| `document` | PDF/Word 解析 | `material_processing` | — |
| `chat` | 意图分类 | 按意图：`question_analysis` / `study_plan` / 教学 | `mistake_classification`（聚合） |

---

## 4. 数据契约交接点

| 交接点 | 格式 | 生产者 | 消费者 |
|--------|------|--------|--------|
| `input_payload` | JSON | Input Layer | Agent / Workflow |
| `DATA_HANDOFF` | JSON in Markdown | `question_analysis` | Memory / Database |
| `MATERIAL_HANDOFF` | JSON in Markdown | `material_processing` | Knowledge Layer |
| `MEMORY_UPDATE` | YAML / JSON 片段 | Agent | `memory/*.md` 运行时区 |

---

## 5. 数据流原则

| 原则 | 说明 |
|------|------|
| **不编造** | 无来源数据不写入 Memory / Database |
| **事实分离** | 用户原文 vs Agent 推断分字段存储 |
| **幂等去重** | 同 `question_id` 更新记录，不重复创建 |
| **读多写少 Knowledge** | 知识库经 Material Processing 审核后写入 |
| **Memory 轻量** | 对话会话可只写 Memory，批量同步 Database |
