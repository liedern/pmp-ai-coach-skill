# Question Coach（题目教练模块）

> **所属产品**：PMP AI Coach（单一主 Skill）  
> **模块类型**：专业能力模块 | **非独立 Skill**

---

## 模块目标

负责 **PMP 情境题的识别、推理、讲解与归档决策**，是用户提交单题后的核心分析引擎。

- 从截图/文字中提取题干与选项
- 按 PMP 五步框架完成考试逻辑分析
- 输出固定结构讲解与 `DATA_HANDOFF` 结构化数据
- 决定是否写入错题记忆（配合 Mistake Coach）

**不负责**：跨题错误模式聚合（→ Mistake Coach）、周计划排期（→ Study Planner）、纯概念科普（→ Training Coach）。

---

## 用户触发场景

| 场景 | 典型用户说法 |
|------|--------------|
| 上传题目截图 | 「帮我看这道题」「分析一下这个截图」 |
| 粘贴题目文字 | 含题干 + A/B/C/D 选项 |
| 提供作答与答案 | 「我选了 A，正确答案是 B」 |
| 仅讲解不保存 | 「讲讲就行，不用保存」 |
| 要求加入错题本 | 「加入错题本」「收录这道题」 |
| 重复做题 | 「这道题我又错了」 |

---

## 输入

| 输入类型 | 来源 | 预处理 |
|----------|------|--------|
| 题目文字 | `inputs/question_input.md` | 结构化题干、选项、用户答案 |
| 题目截图 | `inputs/image_input.md` | OCR → 题目结构化 |
| 用户意图 | `inputs/chat_input.md` | 讲解 / 保存 / 仅分析 |
| 元数据 | 用户说明或消息时间 | 来源、套卷、做题日期 |

**可选上下文**：`memory/mistake_memory.md`（是否重复错题）、`memory/user_profile.md`（讲解深度偏好）。

---

## 输出

| 产出 | 格式 | 说明 |
|------|------|------|
| 固定结构讲解 | Markdown（§7 模板） | 题目信息、正确答案、知识点、逐项分析、记忆规则等 |
| 结构化交接 | `DATA_HANDOFF` JSON | 供 Mistake Coach / Database 消费 |
| 归档决策 | `review_status` | `explain_only` / `wrong` / `needs_review` / `bookmarked` |
| 加工报告（批量时） | — | 本模块单题为主 |

### MVP 实现（`mvp-1`）

| 文件 | 说明 |
|------|------|
| `module.md` | 输入/输出契约、七项结构化分析、下游交接 |
| `output_contract.md` | `QUESTION_OUTPUT` JSON 字段定义 |
| `examples/sample_output.json` | 端到端输出示例 |

---

## 数据依赖

### 工作流（Workflows）

| 文件 | 关系 |
|------|------|
| `workflows/question_analysis.md` | **主执行流程**（必读） |

### 知识库（Knowledge）

| 路径 | 用途 |
|------|------|
| `knowledge/decision_framework/decision_tree.md` | 六步决策链 |
| `knowledge/decision_framework/pmp_priority_rules.md` | 多选项仲裁 |
| `knowledge/decision_framework/predictive_logic.md` | 预测型分支 |
| `knowledge/decision_framework/agile_logic.md` | 敏捷分支 |
| `knowledge/exam/question_patterns.md` | First/Next/Best |
| `knowledge/exam/trap_patterns.md` | 干扰项陷阱 |
| `knowledge/exam/exam_keywords_mapping.md` | 题干关键词 |
| `knowledge/language/*` | 术语与易混概念 |

### 数据模型（Database）

| 文件 | 用途 |
|------|------|
| `database/mistake_schema.md` | 入库字段映射（保存时） |

### 记忆（Memory）

| 文件 | 读写 |
|------|------|
| `memory/mistake_memory.md` | 写（用户确认保存时） |
| `memory/data/mistake_memory.json` | 写（MVP 运行时错题库） |
| `memory/user_profile.md` | 读（个性化讲解） |

### 主 Skill

| 文件 | 关系 |
|------|------|
| `skill.md` §3–§5 | 推理框架、输出格式、错因分类 |

---

## 模块边界

```
用户题目输入
      │
      ▼
Question Coach ──保存──→ Mistake Coach
      │
      └──讲解完成──→ Review Coach（生成单题复习项）
                    Study Planner（纳入计划时）
```
