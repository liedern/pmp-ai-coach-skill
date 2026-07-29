# PMP Terms（核心术语库）

> **用途**：PMP 高频英文术语详解（考试语境 + 场景 + 易混）。全量索引见 `knowledge/terminology/pmp_glossary.md`。
>
> **来源**：【事实】`source_materials/terminology/PMP中英文词组翻译.pdf` + 【推测】考试语境补充
>
> **关联**：`confusing_terms.md`、`keyword_mapping.md`、`synonym_mapping.md`

---

## 挣值管理（EVM）

### Earned Value Management (EVM)（挣值管理）

| 字段 | 内容 |
|------|------|
| **English Term** | Earned Value Management |
| **中文名称** | 挣值管理 |
| **PMP考试含义** | 整合范围、进度、成本绩效，用 EV/PV/AC 及指数分析偏差并预测完工 |
| **关键词** | EV, PV, AC, CPI, SPI, variance, EAC, baseline |
| **常见场景** | 题干给 BAC/EV/AC/PV，问完工预测或是否超支/落后 |
| **易混淆概念** | 与纯财务 NPV/ROI 项目选择区分；PV 在 EVM 中≠ Present Value |
| **Example** | *CPI=0.9* → 成本效率低，每花 1 元只完成 0.9 元预算工作 |

### Planned Value (PV)（计划价值）

| 字段 | 内容 |
|------|------|
| **English Term** | Planned Value (PV) |
| **中文名称** | 计划价值 |
| **PMP考试含义** | 【事实】截至某时点计划完成工作的预算价值；旧称 BCWS |
| **关键词** | planned, budgeted cost of work scheduled, BCWS |
| **常见场景** | 计算 SV=EV−PV；SPI=EV/PV |
| **易混淆概念** | **Present Value（现值）**— 财务折现语境，非 EVM |
| **Example** | *At month 3, PV=$100k* → 计划此时应完成价值 10 万的工作 |

---

## 范围管理

### Validate Scope（确认范围）

| 字段 | 内容 |
|------|------|
| **English Term** | Validate Scope |
| **中文名称** | 确认范围 |
| **PMP考试含义** | 【事实】正式验收可交付成果，获得客户对**范围**的签字认可 |
| **关键词** | Customer Acceptance, Deliverable, Formal Acceptance, Sign-off |
| **常见场景** | 阶段或项目交付完成，问如何让客户正式接受成果 |
| **易混淆概念** | Control Quality（查质量）；Control Scope（管范围变更） |
| **Example** | *The customer formally accepts the deliverable.* → Validate Scope |

### Scope Creep（范围蔓延）

| 字段 | 内容 |
|------|------|
| **English Term** | Scope Creep |
| **中文名称** | 范围蔓延 |
| **PMP考试含义** | 【事实】未批准的范围逐渐扩大 |
| **关键词** | unapproved, additional features, without change request |
| **常见场景** | 客户口头加功能，团队默默执行 |
| **易混淆概念** | Gold Plating（团队主动加功能）；正式 CR 不是蔓延 |
| **Example** | *Features added without approval* → 走变更或拒绝，非默认接受 |

---

## 风险与问题

### Risk Register vs Issue Log

| 字段 | 内容 |
|------|------|
| **English Term** | Risk Register / Issue Log |
| **中文名称** | 风险登记册 / 问题日志 |
| **PMP考试含义** | 【事实】Risk=尚未发生；Issue=已发生 |
| **关键词** | may/might/future vs occurred/already/defect |
| **常见场景** | 供应商**可能**延误 → Risk；已延误两周 → Issue |
| **易混淆概念** | 二者登记册不可混用 |
| **Example** | *The vendor is already two weeks late.* → Issue Log |

---

## 变更管理

### Change Request (CR)（变更请求）

| 字段 | 内容 |
|------|------|
| **English Term** | Change Request |
| **中文名称** | 变更请求 |
| **PMP考试含义** | 【事实】修改基准或可交付成果的正式提议；变更流程起点 |
| **关键词** | CR, CCB, baseline, impact analysis |
| **常见场景** | First 题：变更类首选记录并评估，非立即执行 |
| **易混淆概念** | 直接改基准；口头范围蔓延 |
| **Example** | *Client requests new features* → 先 CR |

---

## 组织环境

### EEF vs OPA

| 字段 | 内容 |
|------|------|
| **English Term** | Enterprise Environmental Factors / Organizational Process Assets |
| **中文名称** | 事业环境因素 / 组织过程资产 |
| **PMP考试含义** | 【事实】EEF=约束（难改）；OPA=资产（可用可更新） |
| **关键词** | EEF: culture, regulations, market; OPA: templates, lessons learned |
| **常见场景** | 新法规出台 → EEF；查历史模板 → OPA |
| **易混淆概念** | 二者均为组织影响，但可控性相反 |
| **Example** | *Use company's risk template* → OPA |

---

## 进度管理

### Fast Tracking vs Crashing

| 字段 | 内容 |
|------|------|
| **English Term** | Fast Tracking / Crashing |
| **中文名称** | 快速跟进 / 赶工 |
| **PMP考试含义** | 【事实】Fast Tracking=并行活动；Crashing=加资源缩工期 |
| **关键词** | parallel vs additional resources, cost increase |
| **常见场景** | 进度落后需压缩工期 |
| **易混淆概念** | 并行→风险增 rework；赶工→成本增 |
| **Example** | *Perform design and build in parallel* → Fast Tracking |

### Resource Leveling vs Resource Smoothing

| 字段 | 内容 |
|------|------|
| **English Term** | Resource Leveling / Resource Smoothing |
| **中文名称** | 资源平衡 / 资源平滑 |
| **PMP考试含义** | 【事实】Leveling 可改关键路径/工期；Smoothing 一般在浮动时间内调整 |
| **关键词** | over-allocated, critical path, float |
| **常见场景** | 资源冲突导致完工日推迟 → Leveling |
| **易混淆概念** | Smoothing 通常不改变项目完工日 |
| **Example** | *Completion date delayed due to resource limits* → Leveling |

---

## 采购

### FFP vs T&M

| 字段 | 内容 |
|------|------|
| **English Term** | Firm Fixed Price (FFP) / Time and Materials (T&M) |
| **中文名称** | 固定总价合同 / 工料合同 |
| **PMP考试含义** | 【事实】FFP 范围明确卖方风险大；T&M 范围不确定短期用 |
| **关键词** | fixed price, well-defined scope vs hourly rates, uncertain scope |
| **常见场景** | 范围清晰选 FFP；探索性工作选 T&M |
| **易混淆概念** | CPFF（成本加费用，买方风险更高） |
| **Example** | *Scope is well defined* → FFP 更合适 |

---

## 团队与冲突

### Collaborate vs Force（冲突策略）

| 字段 | 内容 |
|------|------|
| **English Term** | Collaborate/Problem Solve vs Force/Direct |
| **中文名称** | 合作解决问题 vs 强迫命令 |
| **PMP考试含义** | 【推测】PMI 倾向先 Collaborate（尤其团队冲突 First 题） |
| **关键词** | face-to-face, win-win vs command, emergency |
| **常见场景** | 两名成员争执 → 先合作解决 |
| **易混淆概念** | Compromise（折中）；Smooth（缓和） |
| **Example** | *Team members disagree on approach* → Collaborate before escalate |

---

## 敏捷

### Product Owner vs Scrum Master

| 字段 | 内容 |
|------|------|
| **English Term** | Product Owner / Scrum Master |
| **中文名称** | 产品负责人 / Scrum Master |
| **PMP考试含义** | 【事实】PO=价值与 Backlog 优先级；SM=流程、障碍、教练 |
| **关键词** | value, priority, backlog vs impediment, facilitate, coach |
| **常见场景** | 谁排优先级 → PO；谁清障碍 → SM |
| **易混淆概念** | PM 直接分派任务（敏捷题常错） |
| **Example** | *Who prioritizes the backlog?* → Product Owner |

---

## 绩效数据链

### Work Performance Data vs Information vs Reports

| 字段 | 内容 |
|------|------|
| **English Term** | Work Performance Data / Information / Reports |
| **中文名称** | 工作绩效数据 / 信息 / 报告 |
| **PMP考试含义** | 【事实】Data=原始；Information=分析后；Reports=格式化输出 |
| **关键词** | raw, analyzed, formatted, distribute |
| **常见场景** | 现场测量值 → Data；偏差分析后 → Information |
| **易混淆概念** | 三者顺序不可颠倒 |
| **Example** | *Collect measurements from team* → Work Performance Data |

---

## 扩展

- 全量词条：`knowledge/terminology/pmp_glossary.md`（36 章）
- 加工记录：`knowledge/terminology/import_log.md`
