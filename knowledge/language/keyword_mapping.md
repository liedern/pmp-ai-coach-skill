# Keyword Mapping（题干关键词映射）

> **用途**：题干/选项英文关键词 → 判断动作 → 关联过程/登记册。与 `exam/exam_keywords_mapping.md` 互补，本文件侧重 **language 层** 术语—动作映射。
>
> **调用**：OCR 后扫描、`decision_tree` Step 1–4。

---

## 1. 问法信号词

| English | 中文 | PMP 判断 | 优先动作类型 |
|---------|------|----------|--------------|
| First | 首先 | 找第一步 | analyze, document, meet |
| Next | 下一步 | 流程 +1 | 紧接标准顺序 |
| Best | 最佳 | 多选择优 | 完整路径 + priority_rules |
| Most Appropriate | 最合适 | 情境适配 | 排除过度/不足 |
| Should | 应该 | PMI 推荐行为 | 流程 + 协作 |
| Immediately | 立即 | ⚠️ 陷阱 | 多非 Best |
| NOT / EXCEPT | 否定 | 选例外 | 反转逻辑 |

---

## 2. 过程类关键词

| English Keyword | 中文 | 考试含义 | 关联知识点 |
|-----------------|------|----------|------------|
| Scope Creep | 范围蔓延 | 未批准范围扩大 | Control Scope |
| Change Request | 变更请求 | 正式修改申请 | Integrated Change Control |
| Risk Register | 风险登记册 | 已识别风险 | Risk Management |
| Issue Log | 问题日志 | 已发生问题 | Issue Management |
| Stakeholder Register | 相关方登记册 | 相关方信息 | Stakeholder Management |
| Lessons Learned | 经验教训 | 全周期记录 | Manage Project Knowledge |
| Baseline | 基准 | 批准的计划版本 | Change Control |
| CCB | 变更控制委员会 | 审批变更 | Integrated Change Control |
| WBS | 工作分解结构 | 范围分解 | Create WBS |
| Product Backlog | 产品待办列表 | PO 排序需求池 | Agile Requirements |
| Sprint Backlog | 迭代待办列表 | 本 Sprint 承诺 | Sprint Planning |
| Impediment | 障碍 | 阻碍进展 | Scrum Master |
| Definition of Done | 完成定义 | 增量完成标准 | Agile Quality |

---

## 3. 场景类关键词

| 关键词 | English | 中文理解 | 对应动作 |
|--------|---------|----------|----------|
| Conflict | Conflict | 冲突 | 沟通、合作解决 |
| Resistance | Resistance | 抵触 | 分析原因、参与策略 |
| Unexpected Change | Unexpected Change | 未预期变化 | 评估影响、CR/Backlog |
| New Risk | New Risk | 新风险 | 更新 Risk Register |
| Variance | Variance | 偏差 | 分析原因、纠正/变更 |
| Defect | Defect | 缺陷 | 控制质量、根因分析 |
| Emergency | Emergency | 紧急 | 仍分析；紧急变更流程 |
| Virtual Team | Virtual Team | 虚拟团队 | 沟通计划加强 |

---

## 4. 项目类型信号词

| 类型 | 关键词 |
|------|--------|
| **Predictive** | WBS, baseline, CCB, Gantt, charter, waterfall, phase gate |
| **Agile** | Sprint, iteration, backlog, PO, SM, increment, user story |
| **Hybrid** | 上述两组同时出现 |

---

## 5. 阶段信号词

| 阶段 | 关键词 |
|------|--------|
| Initiating | charter, business case, identify stakeholders |
| Planning | WBS, schedule network, budget, risk identification |
| Executing | direct, manage team, implement |
| Monitoring & Controlling | variance, change request, performance, control |
| Closing | close, lessons learned, final report, release resources |

---

## 6. Agent 扫描顺序

```
1. §1 问法词 → First/Next/Best
2. §4 项目类型
3. §5 阶段
4. §3 场景词 → 问题类型
5. §2 过程词 → 登记册/过程链
```
