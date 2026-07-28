# PMP Exam Keywords Mapping

> **用途**：题干与选项英文关键词 → 中文理解 → 考试判断动作 → 关联知识点的映射表。供 Agent 在 Step 1–5（`decision_tree.md`）中快速识别信号，以及 Terminology Problem 错因分析。
>
> **调用时机**：OCR/粘贴题目后优先扫描本表；与 `question_patterns.md`、`pmp_priority_rules.md` 配合使用。
>
> **原则**：关键词命中 ≠ 唯一答案；须结合项目类型、阶段与问法综合判断。

---

## 1. Process Keywords

过程与工件类关键词：命中后指向对应管理过程或登记册，并触发标准动作链。

| English Keyword | 中文理解 | 考试含义 | 关联知识点 |
|-----------------|----------|----------|------------|
| Scope Creep | 范围蔓延 | 未批准的范围扩大；应拒绝或走变更 | Control Scope / Change Control |
| Gold Plating | 镀金 | 主动超出批准范围添加功能；不应鼓励 | Scope Management |
| Change Request (CR) | 变更请求 | 正式提出修改申请；变更流程起点 | Perform Integrated Change Control |
| Change Control Board (CCB) | 变更控制委员会 | 审查、批准/驳回变更的治理机构 | Integrated Change Control |
| Baseline | 基准 | 范围/进度/成本经批准的版本；不可随意修改 | Baseline Management |
| Work Breakdown Structure (WBS) | 工作分解结构 | 范围的可交付成果分解 | Create WBS |
| Project Charter | 项目章程 | 授权项目；发起人批准 | Develop Project Charter |
| Project Management Plan | 项目管理计划 | PM 制定的整合计划及子计划 | Develop Project Management Plan |
| Risk Register | 风险登记册 | 记录已识别风险及应对 | Identify / Plan Risk Responses |
| Issue Log | 问题日志 | 已发生问题的记录与跟踪 | Manage Project Knowledge / Issue Mgmt |
| Stakeholder Register | 相关方登记册 | 已识别相关方及分析信息 | Identify Stakeholders |
| Lessons Learned Register | 经验教训登记册 | 全周期经验教训记录 | Manage Project Knowledge |
| Requirements Traceability Matrix | 需求跟踪矩阵 | 需求 ↔ 可交付成果 ↔ 测试 | Collect Requirements |
| Quality Metrics | 质量测量指标 | 可量化的质量标准 | Plan Quality Management |
| Definition of Done (DoD) | 完成定义 | 增量/可交付成果的完成标准（敏捷） | Agile Quality |
| Product Backlog | 产品待办列表 | PO 拥有的按价值排序的需求池 | Agile Requirements |
| Sprint Backlog | 迭代待办列表 | 本 Sprint 承诺的工作 | Sprint Planning |
| Procurement Documents | 采购文件 | RFP、RFQ、招标文件等 | Conduct Procurements |
| Contract | 合同 | 具有法律约束的协议 | Control Procurements |
| Earned Value (EV) | 挣值 | 已完成工作的预算价值 | Control Costs / EVM |
| Variance | 偏差 | 实际与计划的差异 | Monitoring & Controlling |
| Corrective Action | 纠正措施 | 纠正已发生偏差 | Direct and Manage Project Work |
| Preventive Action | 预防措施 | 防止未来偏差 | Quality / Risk |
| Defect Repair | 缺陷修复 | 修复不符合项 | Control Quality |
| Configuration Management | 配置管理 | 控制产品/文档版本与变更 | Integrated Change Control |
| Resource Breakdown Structure (RBS) | 资源分解结构 | 按资源类别分解 | Estimate Activity Resources |
| Responsibility Assignment Matrix (RAM) | 责任分配矩阵 | RACI 等角色职责 | Plan Resource Management |
| Statement of Work (SOW) | 工作说明书 | 采购范围描述 | Plan Procurement Management |

### 过程关键词 → 优先动作速查

| 命中关键词 | First 倾向 | 禁止作为 First |
|------------|------------|----------------|
| Change Request / 变更 | 记录 CR → 评估影响 | 直接执行、直接改基准 |
| Risk Register / 新风险 | 更新登记册 → 分析 | 立即上报（无分析） |
| Issue Log / 已发生 | 记录问题 → 纠正 | 写入风险登记册当风险 |
| Scope Creep | 走变更或拒绝 | 默认接受 |
| Baseline 偏离 | 变更控制流程 | PM 私自调整基准 |
| Lessons Learned | 记录（任何阶段） | 仅 Closing 才记录 |

---

## 2. Question Signal Words

题干问法信号词：决定从流程链的哪一步取答案。

| English | 中文 | PMP 判断 | Agent 动作 |
|---------|------|----------|------------|
| **First** | 首先 | 找**第一步**；分析/记录类优先于执行/升级 | 取标准流程 `[0]`；排除终局方案 |
| **Initially** | 最初 | 同 First | 同 First |
| **At the beginning** | 在开始阶段 | 同 First；注意阶段语境 | 结合 Phase 判断 |
| **Next** | 下一步 | **紧接当前状态**的后续一步 | 还原已完成动作 → +1 |
| **Then / Following** | 然后 / 之后 | 同 Next | 同 Next |
| **After … what should** | …之后应做什么 | 同 Next | 解析题干隐含前置动作 |
| **Best** | 最佳 | **多答案择优**；可含完整路径 | 调用 `pmp_priority_rules.md` |
| **Most appropriate** | 最合适 | 情境适配；排除过度/不足反应 | 同 Best，更重适度 |
| **Most likely** | 最可能 | 推断 PMI 倾向行为 | 选符合 P1–P4 原则者 |
| **Should** | 应该 | PMP **推荐**的规范行为 | 流程 + 协作优先 |
| **Recommended** | 建议的 | 同 Should | 同 Should |
| **Primary responsibility** | 首要职责 | 考角色边界（PO/SM/PM） | 查角色表 |
| **LEAST likely** | 最不可能 | **否定选择题**；选最不该做的 | 反转判断 |
| **NOT** | 不 | 选例外项 | 仔细读否定对象 |
| **EXCEPT** | 除了 | 选不符合的一项 | 排除法 |
| **Immediately** | 立即 | ⚠️ **陷阱信号**；多非 Best | 优先怀疑该选项 |
| **Always / Never** | 总是 / 从不 | ⚠️ 绝对化陷阱 | 优先排除 |
| **Without approval** | 未经批准 | ⚠️ 违反流程 | 通常错误 |
| **Ignore / Bypass** | 忽略 / 绕过 | ⚠️ 违反流程 | 通常错误 |

### 问法 × 答案类型矩阵

| 问法 | 优选动作类型 | 常见干扰项 |
|------|--------------|------------|
| First | analyze, document, assess, meet, update register | execute, escalate, close, re-baseline |
| Next | 流程中紧接的下一标准步骤 | 跳步、回到第一步 |
| Best | 分析 + 协作 + 流程完整 | 仅快、仅狠、仅上报 |
| Should | PMI 规范行为 | 政治讨好、个人英雄主义 |

---

## 3. Scenario Keywords

场景类关键词：描述情境矛盾，指向 ECO 领域与首选响应模式。

| 关键词 | English | 中文理解 | 对应动作 | ECO 倾向 |
|--------|---------|----------|----------|----------|
| 冲突 | Conflict | 意见/利益对立 | 面对面沟通 → 合作/妥协解决 | People |
| 抵触 | Resistance | 相关方不支持 | 了解原因 → 调整参与策略 | People |
| 期望不符 | Unrealistic expectations | 期望与交付脱节 | 管理期望、澄清范围 | People |
| 未预期变化 | Unexpected change | 计划外变化 | 评估影响 → 变更/Backlog | Process |
| 新风险 | New risk / identified risk | 刚识别的潜在事件 | 更新 Risk Register → 分析 | Process |
| 已发生问题 | Issue / occurred / defect | 现实已造成影响 | 更新 Issue Log → 纠正 | Process |
| 偏差 | Variance / behind schedule | 绩效偏离计划 | 分析原因 → 纠正/变更 | Process |
| 范围变更 | Scope change | 修改可交付成果 | Change Request / Backlog | Process |
| 质量问题 | Quality problem / defect | 不符合标准 | 控制质量 → 根因分析 | Process |
| 资源不足 | Resource shortage | 缺人/缺技能 | 评估 → 平衡/获取/变更 | Process |
| 虚拟团队 | Virtual team | 分布式协作 | 加强沟通、文化建设 | People |
| 全球团队 | Global team | 跨时区/文化 | 沟通计划、会议协调 | People |
| 合同争议 | Contract dispute | 买卖双方分歧 | 合同条款 + 采购流程 | Process |
| 合规 | Compliance / regulatory | 法规/政策要求 | 优先满足合规 | Business Environment |
| 商业论证 | Business case | 项目价值依据 | 对齐组织战略 | Business Environment |
| 紧急情况 | Emergency / urgent | 时间压力大 | 仍须分析；可走紧急变更 | Process |
| 高管施压 | Executive pressure | 高层要求改范围 | 变更流程，非口头答应 | People + Process |
| Sprint 中断 | Sprint disruption | 迭代内被打断 | PO 协商 Backlog | Agile |
| 技术债 | Technical debt | 短期妥协累积 | Backlog 透明化、重构排期 | Agile |
| 障碍 | Impediment | 阻碍团队进展 | SM 移除障碍 | Agile |

### 场景 → 标准响应链（First 取链首）

```
Conflict          → meet → understand → collaborate resolve → update plan
Resistance        → analyze cause → engage → adjust engagement strategy
New Risk          → register → analyze → plan response
Issue             → log → assign → correct → verify
Unexpected Change → assess impact → CR/Backlog → approve → update → execute
Quality Defect    → inspect → root cause → corrective/preventive
Resource Shortage → assess impact → negotiate → rebalance/CR
```

---

## 4. Easily Confused Terms

易混术语：选项或题干中出现时，必须按下列边界区分，避免 Concept Confusion 错因。

### Risk vs Issue

| 维度 | Risk（风险） | Issue（问题） |
|------|--------------|---------------|
| **时间** | 未来可能发生 | 已经发生 |
| **本质** | 不确定性事件 | 已造成影响的事件 |
| **登记册** | Risk Register | Issue Log |
| **管理目标** | 规划应对、监督 | 纠正、跟踪关闭 |
| **关键词** | future, may, might, uncertain, probability, threat, opportunity | occurred, already, problem, defect, delay happened, impact now |
| **First 动作** | 更新风险登记册 | 记录问题日志 |
| **考试陷阱** | 已发生的延误仍当风险管 | 混淆登记册 |

```
判断口诀：还没发生 → Risk；已经出事 → Issue
```

### Manage vs Monitor

| 维度 | Manage（管理） | Monitor（监督） |
|------|----------------|-----------------|
| **含义** | 主动指导、执行、协调、建设 | 观察、测量、比较、报告 |
| **主动性** | 高：直接采取行动 | 低：收集数据、评估趋势 |
| **典型过程** | Direct and Manage Project Work | Monitor and Control Project Work |
| **关键词** | direct, lead, implement, execute, facilitate, develop team | track, measure, report, compare, trend, variance, watch |
| **考试指向** | 问「怎么做工作」→ Manage | 问「发现偏差后看什么」→ Monitor/Control |
| **陷阱** | 监督题选执行动作 | 执行题只选「观察」 |

### Change Request vs Change

| 维度 | Change Request | Change（已批准变更） |
|------|----------------|----------------------|
| **状态** | 待审请求 | 已批准并待/已执行 |
| **First** | 记录 + 评估 | 更新计划 + 执行 |
| **陷阱** | 未批先改基准 | 把 CR 当已执行 |

### Validate vs Verify

| 维度 | Validate（确认） | Verify（核实） |
|------|------------------|----------------|
| **对象** | 是否满足**业务需求/用户需要** | 是否符合**规格/标准** |
| **问题** | 做对的事吗？ | 把事情做对了吗？ |
| **关联** | Collect Requirements / 用户验收 | Control Quality / 检查 |

### Mitigate vs Transfer vs Accept (Risk)

| 策略 | 含义 | 信号词 |
|------|------|--------|
| **Mitigate** | 降低概率或影响 | reduce, lessen |
| **Transfer** | 转给第三方（保险、合同） | insurance, contract, outsource |
| **Accept** | 知情接受，可建储备 | low impact, cost prohibitive |
| **Avoid** | 消除威胁，改计划 | eliminate, change approach |
| **Exploit** | 确保机会实现 | opportunity, ensure |

### Charter vs Project Management Plan

| 维度 | Project Charter | Project Management Plan |
|------|-----------------|---------------------------|
| **批准人** | 发起人 / Sponsor | PM（发起人可批整体） |
| **内容** | 高层目标、授权、里程碑 | 如何执行的全部子计划 |
| **变更** | 重大变更可能需发起人 | 通过变更控制更新 |
| **陷阱** | PM 批准章程 | 章程当详细计划用 |

### Product Owner vs Scrum Master

| 维度 | Product Owner | Scrum Master |
|------|---------------|--------------|
| **核心** | 价值、Backlog 优先级 | 流程、障碍、教练 |
| **不该做** | 微管实现、替团队估技术方案 | 定优先级、分派任务 |
| **关键词** | value, priority, backlog | impediment, facilitate, coach |

### Corrective vs Preventive Action

| 维度 | Corrective（纠正） | Preventive（预防） |
|------|--------------------|--------------------|
| **针对** | 已发生的偏差/缺陷 | 潜在的 future 问题 |
| **时机** | 问题已出现 | 问题未出现前 |
| **Best 题** | 解决当前 | 避免再发时预防更优 |

### Qualitative vs Quantitative Risk Analysis

| 维度 | Qualitative | Quantitative |
|------|-------------|--------------|
| **方法** | 概率影响矩阵、专家判断 | 蒙特卡洛、EMV、决策树 |
| **输出** | 优先级排序 | 数值化影响、储备计算 |
| **顺序** | 通常先于 Quantitative | 在 Qualitative 之后 |

### Lag vs Lead

| 维度 | Lag（滞后） | Lead（提前） |
|------|-------------|--------------|
| **含义** | 后继活动推迟开始 | 后继活动提前开始 |
| **示例** | FS + 2 days | SS with negative lag / 快速跟进 |
| **陷阱** | 与 Float 混淆 | 与 Crashing 混淆 |

---

## 5. Agent 扫描流程

```
1. 扫描 Question Signal Words（§2）→ 确定 First/Next/Best
2. 扫描 Scenario Keywords（§3）→ 确定问题类型 + ECO
3. 扫描 Process Keywords（§1）→ 确定过程链与登记册
4. 扫描 Easily Confused Terms（§4）→ 排除概念混淆选项
5. 输出命中表 + 跳转 decision_tree / pmp_priority_rules
```

### 输出示例

```markdown
**关键词命中**
- 问法：First（§2）
- 场景：Conflict（§3）→ People
- 过程：无 CR，非变更题
- 易混：非 Risk（无 future 信号）→ 按冲突处理
→ 优先：面对面沟通合作解决（Collaborate Before Escalate）
```
