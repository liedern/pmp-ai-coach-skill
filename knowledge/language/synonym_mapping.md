# Synonym Mapping（同义词映射）

> **用途**：PMP 题干与教材中同一概念的不同英文表述映射到**标准术语**，避免 Terminology Problem 漏识别。
>
> **规则**：映射到 `pmp_terms.md` 中的标准 English Term。

---

## 1. 过程与同义表述

| 标准术语 | 同义词 / 变体 | 中文 |
|----------|---------------|------|
| Validate Scope | scope validation, formal acceptance of deliverables, client sign-off on scope | 确认范围 |
| Control Quality | quality control, inspect deliverables, QC | 控制质量 |
| Manage Quality | quality assurance, QA, quality audit | 管理质量 |
| Perform Integrated Change Control | integrated change control, change control process, CCB review | 实施整体变更控制 |
| Direct and Manage Project Work | execute project work, manage project execution | 指导与管理项目工作 |
| Monitor and Control Project Work | monitor work, track performance | 监控项目工作 |
| Identify Risks | risk identification, identify threats and opportunities | 识别风险 |
| Manage Stakeholder Engagement | engage stakeholders, stakeholder management | 管理相关方参与 |

---

## 2. 工件与同义表述

| 标准术语 | 同义词 / 变体 |
|----------|---------------|
| Risk Register | risk log, register of risks |
| Issue Log | issue register, problem log |
| Change Log | change register, log of changes |
| Stakeholder Register | stakeholder list |
| Lessons Learned Register | lessons learned log, LL register |
| Product Backlog | backlog, prioritized backlog |
| Sprint Backlog | iteration backlog |

---

## 3. 角色与同义表述

| 标准角色 | 同义词 / 变体 |
|----------|---------------|
| Product Owner | PO, product owner role |
| Scrum Master | SM, agile coach (语境为 Scrum 时) |
| Project Sponsor | sponsor, executive sponsor, initiator |
| Project Manager | PM, project leader (矩阵语境注意边界) |

---

## 4. 策略与同义表述

| 标准策略 | 同义词 / 变体 |
|----------|---------------|
| Mitigate | reduce risk, lessen impact |
| Transfer | insurance, outsource risk, warranty |
| Accept | tolerate risk, passive acceptance |
| Avoid | eliminate threat, change plan to avoid |
| Exploit | ensure opportunity |
| Share | partner on opportunity |

---

## 5. 问法同义表述

| 标准问法 | 同义题干表述 |
|----------|--------------|
| First | initially, at the beginning, should do first, what to do before |
| Next | then, following, afterward, what to do after |
| Best | most effective, greatest benefit, optimal |
| Most Appropriate | most suitable, most likely (in PMI context) |

---

## 6. 中英混合常见写法

| 用户可能输入 | 映射标准术语 |
|--------------|--------------|
| 范围确认 | Validate Scope |
| 质量控制 | Control Quality |
| 变更控制 | Perform Integrated Change Control |
| 风险登记册 | Risk Register |
| 问题日志 | Issue Log |
| 产品负责人 | Product Owner |

---

## 7. 挣值旧称映射（来源 PDF 第一章）

| 标准术语 | 旧称 / 变体 | 中文 |
|----------|-------------|------|
| AC | ACWP, Actual Cost of Work Performed | 实际成本 |
| EV | BCWP, Budgeted Cost of Work Performed | 挣值 |
| PV | BCWS, Budgeted Cost of Work Scheduled | 计划价值（EVM） |

---

## 8. 缩写歧义消歧

| 缩写 | 语境 A → 标准术语 | 语境 B → 标准术语 |
|------|-------------------|-------------------|
| PV | EVM: Planned Value | Finance: Present Value |
| FF | Schedule: Free Float | Dependency: Finish-to-Finish |
| RBS | Risk: Risk Breakdown Structure | Resource: Resource Breakdown Structure |
| SM | Scrum Master | 非 Standard（避免与 Subject Matter 混淆） |

---

## 9. 冲突策略同义表述（PDF 第十七章）

| 标准策略 | 同义词 |
|----------|--------|
| Collaborate/Problem Solve | collaborate, problem solving, win-win |
| Compromise/Reconcile | compromise, reconcile |
| Smooth/Accommodate | smooth, accommodate |
| Force/Direct | force, direct, command |
| Withdraw/Avoid | withdraw, avoid, retreat |

---

## 10. 合同类型缩写（PDF 第十四章）

| 标准术语 | 缩写变体 |
|----------|----------|
| Firm Fixed Price | FFP |
| Fixed Price Incentive Fee | FPIF |
| Cost Plus Fixed Fee | CPFF |
| Cost Plus Incentive Fee | CPIF |
| Cost Plus Award Fee | CPAF |
| Time and Materials | T&M, T and M |

---

## Agent 使用规则

1. OCR/用户输入先归一化为标准 English Term
2. 同义词命中后跳转 `pmp_terms.md` 完整条目
3. 歧义同义词（如 "quality control" 可能指 Control Quality 或口语 QC）→ 结合场景词 disambiguate
4. 旧题库可能用 ACWP/BCWP/BCWS → 映射为 AC/EV/PV
