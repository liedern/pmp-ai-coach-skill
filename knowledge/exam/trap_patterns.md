# Trap Patterns（陷阱模式）

> **用途**：PMP 情景题常见错误选项模式与干扰项套路。供 Agent 在 Step 6 排除选项及讲解「为何看似正确但不是最佳」。
>
> **关联**：`exam/exam_keywords_mapping.md`、`decision_framework/pmp_priority_rules.md`、`language/confusing_terms.md`

---

## 1. 设计原则

PMP 干扰项通常**部分正确**，违反的是：

- 决策**顺序**（First vs Best）
- **优先级原则**（Analysis / Collaborate / Process）
- **角色边界**（PO / SM / PM）
- **概念边界**（Risk vs Issue、Validate vs Control Quality）

---

## 2. 通用陷阱模式

| 模式 ID | 名称 | 描述 | 排除信号 |
|---------|------|------|----------|
| T01 | 过早行动 | 未分析、未记录即执行 | immediately, without assessing |
| T02 | 过早升级 | 第一步找发起人/高管 | escalate, sponsor, senior management (无协作前提) |
| T03 | 绕过流程 | 直接改基准、口头接受变更 | bypass, ignore process, update baseline directly |
| T04 | First 选 Best | 问 First 选长期最优 | 选项为完整解决方案非第一步 |
| T05 | 绝对化 | always / never / all / must | 极端措辞 |
| T06 | 角色越权 | SM 定优先级、PM 批章程 | 角色与动作不匹配 |
| T07 | 登记册混淆 | Risk/Issue、Charter/Plan | 已发生写入 Risk Register |
| T08 | 过度反应 | 小题大做、全员加班 | 不符合 Most Appropriate |
| T09 | 反应不足 | 忽视问题、不记录 | 无登记、无沟通 |
| T10 | 文档过度 | 敏捷情境冗长文档 | 应用面对面沟通 |
| T11 | 文档不足 | 预测型无书面变更 | 口头变更无 CR |
| T12 | 镀金/蔓延 | 主动加功能或接受未批范围 | gold plating, scope creep 接受 |

---

## 3. 按问法分类的陷阱

### 3.1 First 题陷阱

| 看似正确 | 实际错误原因 | 应选类型 |
|----------|--------------|----------|
| 立即执行变更 | 跳过 CR 与评估 | 记录 + 评估影响 |
| 上报发起人 | 跳过团队协作 | 面对面沟通 |
| 更新项目管理计划 | 尚未批准变更 | 提交变更请求 |
| 关闭项目 | 问题未解决 | 分析/纠正 |

### 3.2 Best 题陷阱

| 看似正确 | 实际错误原因 |
|----------|--------------|
| 仅沟通无跟进 | 不完整 |
| 仅执行无分析 | 违反 Analysis Before Action |
| 仅处罚无根因 | 违反 Root Cause Before Solution |

### 3.3 NOT / EXCEPT 题陷阱

- 选「最像正确答案」的一项 → 应选**最不符合 PMI 逻辑**的
- 忽略否定词 NOT、LEAST

---

## 4. 按项目类型分类

### 4.1 Predictive 陷阱

| 陷阱 | 说明 |
|------|------|
| PM 改基准 | 须经 CCB |
| 章程当计划 | 批准人与粒度不同 |
| 经验教训仅收尾 | 全周期收集 |
| 风险当问题 | 已发生用 Issue Log |

### 4.2 Agile 陷阱

| 陷阱 | 说明 |
|------|------|
| PM 分派任务 | 团队自组织 |
| Sprint 中随意加需求 | 经 PO / Backlog |
| Daily Scrum 汇报会 | 15 分钟同步，非向 PM 汇报 |
| Review 与 Retro 混淆 | Review=展示；Retro=改进 |

### 4.3 Hybrid 陷阱

| 陷阱 | 说明 |
|------|------|
| 迭代内走 CCB 改 Sprint Goal | 分清外部治理与内部迭代 |
| 用瀑布计划管 Backlog 优先级 | PO 负责价值排序 |

---

## 5. 按 ECO 领域分类

| 领域 | 常见陷阱 |
|------|----------|
| **People** | 冲突即升级；忽视相关方参与计划 |
| **Process** | 跳过变更；混淆 Validate/Control Quality |
| **Business Environment** | 忽视合规；商业论证与项目脱节 |

---

## 6. PMP 答题优先级（排除顺序）

当 2+ 选项看似合理时，按以下顺序保留：

```
1. Analysis Before Action      → 保留「先分析/记录」
2. Collaborate Before Escalate → 保留「先沟通」
3. Follow Process Before Changing → 保留「走流程/Backlog」
4. Root Cause Before Solution  → 保留「找根因」
5. Preventive Before Corrective → 防再发场景保留预防
6. Team Participation          → 保留团队参与
```

详见 `decision_framework/pmp_priority_rules.md`。

---

## 7. 选项措辞红旗词

优先**怀疑**含以下词的选项（非绝对排除）：

| 红旗词 | 原因 |
|--------|------|
| immediately | 常跳过分析 |
| always / never | 绝对化 |
| without approval | 违反流程 |
| ignore the process | 明示违规 |
| fire / replace the team | 过度反应 |
| add to scope without CR | 范围蔓延 |

---

## 8. Agent 排除流程

```
对每个选项：
  1. 标匹配陷阱模式 T01–T12
  2. 标违反的 Priority Rule P1–P6
  3. 标易混概念（查 confusing_terms）
  4. 若问法 First，排除非链首动作
  5. 剩余选项用 priority_rules 仲裁
```

---

## 9. 讲解输出模板

```markdown
### 干扰项分析
- **A**：触犯 T02 过早升级 — 应先与团队沟通（Collaborate Before Escalate）
- **C**：触犯 T07 — 已发生问题应记入 Issue Log，非 Risk Register
```
