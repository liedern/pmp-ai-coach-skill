# Question Patterns（情景题规律）

> **用途**：定义 PMP 情景题的识别规律、问法判断与项目类型分流。供 Agent 在 `decision_tree` Step 1、Step 5 调用。
>
> **关联**：`trap_patterns.md`、`decision_framework/decision_tree.md`、`language/keyword_mapping.md`

---

## 1. 情景题结构

典型 PMP 情境题包含：

| 组件 | 说明 |
|------|------|
| **场景描述** | 项目类型线索、组织环境、约束 |
| **冲突/事件** | 变更、风险、冲突、质量、相关方等 |
| **问法** | First / Next / Best / Most Appropriate |
| **四个选项** | 常含 1 正确 + 2–3 部分正确干扰项 |

---

## 2. First / Next / Best / Most Appropriate 判断

### 2.1 First（首先）

| 项目 | 规则 |
|------|------|
| **含义** | 当前情境下**最先**采取的一步 |
| **信号词** | first, initially, at the beginning, should do first |
| **优选** | 分析、记录、查阅登记册、面对面沟通 |
| **排除** | 最终方案、收尾、未经分析的升级或执行 |

**标准链首动作**：

| 问题类型 | First 倾向 |
|----------|------------|
| 变更 | 记录 Change Request / 评估影响 |
| 新风险 | 更新 Risk Register |
| 已发生问题 | 记录 Issue Log |
| 团队冲突 | 面对面沟通 |
| 相关方抵制 | 了解原因、沟通 |
| 敏捷新需求 | PO 纳入 Backlog |

### 2.2 Next（下一步）

| 项目 | 规则 |
|------|------|
| **含义** | 题干隐含「已完成某动作」后的**紧接一步** |
| **信号词** | next, then, following, after … |
| **方法** | 还原状态机 → 当前节点 +1 |

**示例状态链（变更）**：

```
记录 CR → 评估影响 → 提交 CCB → 批准 → 更新计划 → 执行
```

### 2.3 Best（最佳）

| 项目 | 规则 |
|------|------|
| **含义** | 综合最优，可含多步逻辑的整体最佳 |
| **信号词** | best, most effective, optimal |
| **方法** | 完整路径 + `pmp_priority_rules` 仲裁 |
| **注意** | 可与 First 答案相同（若问法不同） |

### 2.4 Most Appropriate（最合适）

| 项目 | 规则 |
|------|------|
| **含义** | 情境最贴合，排除过度与不足反应 |
| **信号词** | most appropriate, most suitable |
| **方法** | 适度原则；Hybrid 看焦点侧 |

### 2.5 Should（应该）

| 项目 | 规则 |
|------|------|
| **含义** | PMI 规范推荐的应有行为 |
| **信号词** | should, recommended |
| **方法** | 流程 + 协作，非政治讨好 |

---

## 3. Predictive / Agile / Hybrid 判断

### 3.1 信号对照

| 类型 | 强信号（≥2 项高置信） |
|------|------------------------|
| **Predictive** | WBS, baseline, CCB, Gantt, charter, change log, phase gate, EVM |
| **Agile** | Sprint, backlog, PO, SM, iteration, increment, user story, DoD |
| **Hybrid** | 两组信号同时出现 |

### 3.2 分流规则

```
若 Hybrid：
  问治理/合同/基准 → Predictive 逻辑（predictive_logic.md）
  问迭代/Backlog/Sprint → Agile 逻辑（agile_logic.md）
```

### 3.3 同一题型的类型差异

| 情境 | Predictive | Agile |
|------|------------|-------|
| 需求变更 | Change Request → CCB | PO → Product Backlog |
| 团队障碍 | PM 协调 / 资源经理 | SM 移除 impediment |
| 优先级 | 章程/计划 | PO 排序 Backlog |

---

## 4. 常见错误选项模式

| 模式 | 表现 | 识别 |
|------|------|------|
| **过早执行** | 立即改计划、立即交付 | 无 CR / 无分析 |
| **过早升级** | 第一步找 sponsor | 无协作前提 |
| **角色错误** | SM 排优先级 | 查角色表 |
| **登记册错** | 已发生写 Risk Register | Risk vs Issue |
| **过程错** | 验收题选 Control Quality | Validate vs Control |
| **First/Best 混** | First 题选终局方案 | 对照问法 |
| **绝对化** | always / never | 红旗词 |
| **敏捷瀑布混** | Sprint 内走 CCB 改目标 | Hybrid 分流错误 |

详见 `trap_patterns.md`。

---

## 5. PMP 答题优先级

多选项均合理时，按优先级保留（高覆盖低）：

| 优先级 | 原则 |
|--------|------|
| P1 | Analysis Before Action |
| P2 | Collaborate Before Escalate |
| P3 | Follow Process Before Changing |
| P4 | Root Cause Before Solution |
| P5 | Preventive Before Corrective |
| P6 | Team Participation Before PM Decision |

完整规则：`decision_framework/pmp_priority_rules.md`。

---

## 6. Agent 情景题分析顺序

```
1. 扫描问法（§2）→ First / Next / Best
2. 扫描项目类型（§3）→ Predictive / Agile / Hybrid
3. 识别问题类型 → 变更 / 风险 / 冲突 / 质量 …
4. 加载 decision_tree Step 2–6
5. 用 §4 模式排除干扰项
6. 用 §5 优先级仲裁剩余选项
```

---

## 7. 输出示例

```markdown
**情景题规律命中**
- 问法：First
- 类型：Predictive（CCB, baseline）
- 问题：变更
- First 链首：记录变更请求并评估影响
- 排除：A（过早升级 T02）、D（直接执行 T01）
```
