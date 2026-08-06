# Question Capture Workflow（单题采集）

> **版本**：`mvp-1.1`（Answer Validation v0.1.1 采集侧）  
> **定位**：真实学习场景的**入口工作流**——用**最少用户输入**完成结构化采集，再交给 `question_analysis.md` 做 PMP 讲解与 `DATA_HANDOFF` 写入。  
> **不新增** Memory 文件、数据库表或实体类型；`Question` 仍为逻辑实体（见 `database/question_schema.md` §1 MVP）。

---

## 1. Purpose

| 目标 | 说明 |
|------|------|
| 适配真实场景 | 截图 OCR 残缺、只记了「我选 B」、事后补正确答案与解析 |
| 最少输入 | 按 **采集档位 L0–L3** 递进；能判则不再追问 |
| 统一入口 | 文字 / 图片 / 混贴「题干+解析」均先走本流程再分析 |
| 复用三分流 | 与 `question_analysis.md` §6 一致：History 必有作答；答错自动 Mistake；收藏仅 Bookmark |

```
用户输入（任意形态）
        │
        ▼
Question Capture（本文件）── 结构化 + 缺口清单 + question_id
        │
        ▼
Question 创建/匹配（逻辑 Question，无 questions.json）
        │
        ▼
Question Analysis Workflow ── 讲解 + 错因 + DATA_HANDOFF
        │
        ├─► question_history.json（有 user_answer 或 result 可判）
        ├─► 答错 → mistake_memory.json → weak_points 聚合
        └─► 用户说收藏 → bookmark_memory.json
```

---

## 2. Trigger（触发条件）

以下任一满足即进入 **Question Capture**（随后必接 Question Analysis，除非用户 `force_skip`）：

| 信号 | 示例 |
|------|------|
| 题目截图 | App 练习页、拍照 |
| 粘贴题干/选项 | 含 A/B/C/D |
| 补录错题 | 「我选 B，正确 C，练习 8/180」 |
| 带解析录入 | 粘贴培训机构解析全文 |
| 最短指令 | 「录入：」+ 任意片段（见 §4） |

**不触发**：纯概念问法、无题可判 → `Training Coach`（`skill.md` §2.4）。

---

## 3. 采集档位（最少输入）

Agent **从已有信息推断档位**，仅对**阻断写入**的缺口提问（一次最多 3 项）。

| 档位 | 用户最少提供 | Agent 行为 | 能否写 History | 能否写 Mistake |
|------|----------------|------------|:--------------:|:--------------:|
| **L0** | 截图 **或** 题干+选项 | OCR/切分；`user_answer`/`correct_answer` 待确认 | 仅当之后补全答案 | 否（无对错） |
| **L1** | L0 + **我的答案**（字母或选项原文） | 无标答则【推测】+ `needs_review`；有作答写 History | ✅ | 仅当标答已知且判错 |
| **L2** | L1 + **正确答案**（或官方解析中含标答） | 完整对错；答错自动 Mistake（对 **adjudication**） | ✅ | 答错 ✅（见 Analysis §5.5） |
| **L3** | L2 + **解析**（可选） | 优先采用【事实】解析，减少推测讲解 | ✅ | 答错 ✅ |

**收藏**：任意档位均可；用户说「收藏/已收藏」→ 仅 `bookmark_memory.json`，**不**替代 Mistake。

**用户显式**：「只要讲解不保存」→ `force_skip=true`（不写 Mistake；有作答仍可写 History，见 `question_analysis` §1）。

---

## 4. 用户最少输入模板（推荐复制）

### 4.1 一行式（L2，推荐）

```text
录入 | 我选{B} 正确{C} | 练习 8/180
（下方粘贴题干与选项，或附图）
```

### 4.2 两行式（L1，仅记了错选）

```text
我选 B
（题干 + 选项）
```

### 4.3 仅截图（L0→L1）

上传图片；若 App 已标红错项，OCR 出 `user_answer`；**一句确认**：「正确答案是 A 吗？」仅在 OCR 未识别标答时询问。

### 4.4 字段别名（解析粘贴区）

识别下列标题为结构化字段（中英文均可）：

| 别名 | 映射字段 |
|------|----------|
| 我的答案 / 用户答案 / 我选 | `user_answer` |
| 正确答案 / 标答 / 答案 | `platform_answer`（题库侧；非判题唯一依据） |
| 解析 / 分析 / Explanation | `official_explanation` |
| 来源 / 题库 | `source` |
| 练习 x/180、第 x 题 | `exam_set` / 题号元数据 |

---

## 5. Capture Pipeline（执行步骤）

### Step 5.1 归一化输入

```
raw_input（text | image | mixed）
    │
    ├─ image → OCR → ocr_text（置信度 high/medium/low）
    ├─ 切分：题干 | options | 答案区 | 解析区
    └─ 标记【事实】/【推测】/【待确认】（同 question_analysis §0）
```

输出内部对象 **`CAPTURE_RECORD`**（见 §8）。

### Step 5.2 Question 创建 / 匹配

**不写入**独立 `questions.json`（MVP 无题库表文件）。

| 动作 | 规则 |
|------|------|
| 匹配 | 读 `question_history.json` / `mistake_memory.json`：题干归一化 + 选项指纹（A–D 文本） |
| 命中 | 复用已有 `question_id` |
| 新建 | 分配 `question_id`：`q_{语义短 slug}_{序号}`（小写、下划线，与现有 `memory/data` 风格一致） |
| 逻辑 Question | 在 `CAPTURE_RECORD` 与后续 `DATA_HANDOFF` 中携带 `question_text`、`options`、`platform_answer`（及兼容键 `correct_answer`=采集标答）、`source`、`exam_set` |

### Step 5.3 判定 `result`（预检；最终以 Analysis §5.5 `adjudication_answer` 为准）

```text
normalize(user_answer) == normalize(adjudication_answer)
  AND both non-null     → result = correct
normalize(user_answer) != normalize(adjudication_answer)
  AND both non-null     → result = wrong
仅一方为 null           → result = unknown
user_answer 为 null     → 不写 Mistake
```

Capture 阶段若尚未有 `adjudication_answer`，可用 `platform_answer` **仅作预检**；写入 History 前必须由 Question Analysis 完成 AEL。

```text
（预检，可选）normalize(user_answer) vs normalize(platform_answer)
```

`normalize`：去空格、统一为大写字母；选项原文匹配时映射到 A–D。

### Step 5.4 路由写入（Memory 不变）

| 条件 | `question_history.json` | `mistake_memory.json` | `bookmark_memory.json` |
|------|-------------------------|------------------------|-------------------------|
| 有 `user_answer` 且非 `force_skip` 讲解-only | 追加 `history_id` | — | — |
| `result = wrong`（对 adjudication）且 `error_type`+`error_reason` 完整且 `decision_rules` §0.2 通过 | 已写 | **自动追加/更新** Mistake | — |
| 争议：用户=Coach ≠ 平台 | 已写 `answer_disputed` | **不写** | — |
| `result = correct` | 已写 | **不写** | — |
| 用户「收藏」 | 不变 | 不变 | 追加 `bookmark_id` |
| `result = unknown` | 可写 `result: unknown` | **不写** | — |

答错后 **WeakPoint / learning_state**：仍由 `mistake_classification.md` 聚合，本流程不重复实现。

### Step 5.5 交接 Question Analysis

将 `CAPTURE_RECORD` 作为 `question_analysis.md` §2 的预填充输入，执行 §3–§8（讲解 + `DATA_HANDOFF`）。

**禁止**：在 Capture 阶段单独询问「是否保存错题」。

---

## 6. OCR / 截图规则

| ocr_confidence | 行为 |
|----------------|------|
| high | 直接填 `question_text` / `options` |
| medium | 填字段 + 输出【待确认】列表（≤3 项） |
| low | 仅保存已识别片段；**不**自动写 Mistake；提示用户补文字或重拍 |

从练习 App 截图优先读取：

- 进度条 `x/180` → `exam_set`
- 标红选项 → `user_answer`
- 蓝框/解析页标答 → `platform_answer`（【事实】；须经 Analysis 生成 `coach_answer`）

---

## 7. 与 `question_analysis` 的分工

| 阶段 | 本文件 Capture | `question_analysis` |
|------|----------------|---------------------|
| 输入形态、最少追问 | ✅ | 消费 Capture 结果 |
| PMP 推理、选项逐项分析 | — | ✅ |
| `error_type` / `exam_domain` | — | ✅ §5 |
| Answer Evaluation（AEL） | — | ✅ §5.5 |
| 固定输出模板 §7 | — | ✅ |
| `DATA_HANDOFF` JSON | — | ✅ §8 |
| Memory 物理写入 | 决策矩阵 §5.4 | 与 Mistake 工作流一致 |

---

## 8. CAPTURE_RECORD（内部交接 JSON）

Capture 完成后、分析前，Agent 内部保持一致（可附在日志，**不要求**用户可见）：

```json
{
  "workflow": "question_capture",
  "version": "mvp-1.0",
  "capture_level": "L2",
  "question_id": "q_change_mgmt_plan_first_storage_001",
  "question_action": "create | match",
  "question_text": "…",
  "options": { "A": "…", "B": "…", "C": "…", "D": "…" },
  "user_answer": "B",
  "platform_answer": "A",
  "correct_answer": "A",
  "official_explanation": null,
  "source": "培训机构题库",
  "exam_set": "练习 8/180",
  "ocr_confidence": "high",
  "result": "wrong",
  "data_routing": {
    "write_history": true,
    "write_mistake": true,
    "write_bookmark": false
  },
  "pending_fields": [],
  "force_skip_mistake": false
}
```

`write_mistake` 仅在 `result=wrong` 且分析阶段产出 `error_type`+`error_reason` 后为 `true`。

---

## 9. 输出给用户（Capture 摘要）

在完整「PMP 题目分析」之前或之后，用**短块**确认采集结果（可选，建议 L0/L1）：

```markdown
## 采集结果

- **question_id**：`q_…`（新建 / 匹配已有）
- **档位**：L2
- **对错**：错（B ≠ A）
- **将写入**：History ✅ · Mistake ✅（答错自动）· 收藏：否
- **仍待确认**：无
```

---

## 10. 边界与错误处理

| 情况 | 处理 |
|------|------|
| 选项不足 2 项 | 不判对错；`needs_review`；可写 History `skipped` |
| 标答【推测】且用户答错 | 可入库 Mistake，`confidence_level=low`（见 question_analysis §6.3） |
| 与 Mistake 重复 `question_id` 再错 | 新 History + 更新 Mistake `repeated_count` |
| 用户答对但要求收藏 | History + Bookmark，无 Mistake |
| 批量 PDF/多题 | **不走**本流程；用 `material_processing.md` §5 |

---

## 11. 引用

| 文档 | 关系 |
|------|------|
| `inputs/question_capture_input.md` | 用户侧最少输入说明 |
| `inputs/question_input.md` / `image_input.md` | 输入类型识别 |
| `workflows/question_analysis.md` | 分析 + Handoff + 输出模板 |
| `workflows/mistake_classification.md` | 答错入库与 WeakPoint |
| `database/question_schema.md` | Question / History / Bookmark |
| `database/schema_overview.md` | 三类记忆 P0 |
