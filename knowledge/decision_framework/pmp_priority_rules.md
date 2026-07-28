# PMP Priority Rules

> **用途**：多答案场景下的优先级判断规则。当多个选项「看起来都对」时，Agent 按本文件原则与对照表选出**最佳**答案。
>
> **调用时机**：`decision_tree.md` Step 6 过滤后仍有 2+ 选项合理；或问法为 Best / Most Appropriate。
>
> **关联文件**：`decision_tree.md`、`predictive_logic.md`、`agile_logic.md`

---

## Purpose

PMP 情境题常设置多个**部分正确**的干扰项。本文件用于帮助 Agent 在多个选项都合理时，判断**最佳答案**（Best / Most Appropriate），而非「第一个能想到的动作」或「最激进的动作」。

**核心任务**：

- 在并列合理选项中，按 PMP 考试逻辑排出优先级
- 识别「正确但不优先」与「看似高效但违规」的选项
- 输出选择依据：命中哪条 Priority Rule、排除其他项的理由

**不适用**：

- 仅一个选项符合基本逻辑 → 直接选，无需优先级仲裁
- 问法为 First / Next → 优先用 `decision_tree.md` 流程顺序定位，再用本文件打破平局

---

## Core Principles

以下原则**按优先级从高到低排序**。高优先级原则覆盖低优先级；冲突时选更符合高优先级原则的选项。

### 1. Analysis Before Action

**先分析，再行动。**

| 项目 | 说明 |
|------|------|
| **含义** | 在采取行动、拒绝、升级或变更之前，先收集信息、评估影响、查阅登记册/计划 |
| **适用** | 问题出现、风险发生、冲突出现、需求变化 |
| **优先动作** | 查阅项目文件、评估影响、记录到登记册/日志、根因分析 |
| **靠后动作** | 立即执行、立即拒绝、立即上报、立即改计划 |
| **关键词** | analyze, assess, evaluate, understand, review, identify impact |

### 2. Collaborate Before Escalate

**优先沟通协作，再升级。**

| 项目 | 说明 |
|------|------|
| **含义** | 先与团队、相关方、PO、卖方等直接沟通协商，再上报发起人/管理层 |
| **适用** | 团队分歧、相关方抵制、冲突、需要额外资源 |
| **优先动作** | 面对面会议、合作解决问题、协商、澄清期望 |
| **靠后动作** | 第一时间找发起人、高管、CCB（无前置沟通时） |
| **关键词** | meet, discuss, collaborate, negotiate, facilitate |
| **例外** | 题干已明确「多次沟通无效」或「超出 PM 权限且已尝试协作」→ 升级可成为 Best |

### 3. Follow Process Before Changing

**遵循项目管理流程，不直接修改基准。**

| 项目 | 说明 |
|------|------|
| **含义** | 变更、基准调整、范围修改须走正式流程（变更控制 / Backlog），不得绕过 |
| **适用** | 范围、进度、成本、基准、Sprint 承诺变更 |
| **优先动作** | 变更请求、CCB 审批、PO 更新 Backlog、更新计划后执行 |
| **靠后动作** | PM 直接改基准、口头接受范围蔓延、Sprint 内静默加需求 |
| **关键词** | change request, CCB, baseline, backlog, approved |

### 4. Root Cause Before Solution

**先找根因，再解决问题。**

| 项目 | 说明 |
|------|------|
| **含义** | 对反复缺陷、绩效偏差、冲突根源，先分析原因再实施纠正 |
| **适用** | 质量缺陷、进度延误、团队摩擦、相关方不满 |
| **优先动作** | 根因分析、鱼骨图、5 Why、回顾检视 |
| **靠后动作** | 仅打补丁、换人、加班、处罚（未找根因） |
| **关键词** | root cause, why, underlying reason, analyze before fix |

### 5. Preventive Action Before Corrective Action

**预防优先于纠正。**

| 项目 | 说明 |
|------|------|
| **含义** | 在已出问题后，Best 答案可含纠正；但若问「最能避免再发」，选预防措施 |
| **适用** | 质量事故后、风险成真后、重复错误 |
| **优先动作** | 更新过程、培训、改流程、风险应对、过程改进 |
| **靠后动作** | 仅修复当前缺陷、仅返工、仅道歉 |
| **关键词** | preventive, process improvement, avoid recurrence, update procedure |

### 6. Team Participation Before PM Decision

**鼓励团队参与，而不是项目经理单独决定。**

| 项目 | 说明 |
|------|------|
| **含义** | 估算、方案选择、改进措施宜由团队参与；PM 整合而非独断 |
| **适用** | 估算、技术方案、回顾改进、冲突解决、自组织团队 |
| **优先动作** | 团队研讨、集体估算、Retrospective、邀请团队提出方案 |
| **靠后动作** | PM 单独拍板、PM 直接分派（敏捷语境）、绕过团队 |
| **关键词** | team input, collaborative, self-organizing, facilitate |

### 原则冲突仲裁

```
多选项均「合理」时：
  → 从上往下匹配，命中更高优先级原则者胜出
  → 若问法为 First，「分析/记录」类通常优于「解决/升级」类
  → 若问法为 Best，允许组合路径，但不得跳过未满足的高优先级原则
```

---

## PMP Decision Priority Table

格式：**场景 | 错误动作（看似合理） | 优先动作 | PMP 原则**

### 变更

| 场景 | 错误动作（看似合理） | 优先动作 | PMP 原则 |
|------|----------------------|----------|----------|
| 客户要求增加功能 | 为保关系立即安排开发 | 记录变更请求，评估对基准影响 | Analysis Before Action + Follow Process |
| 发现范围偏离基准 | PM 直接调整进度基准赶工 | 提交变更请求，走 CCB 审批 | Follow Process Before Changing |
| 紧急变更压力 | 先执行后补批 | 评估影响 → 紧急变更流程 → 批准 → 执行 | Follow Process + Analysis |
| Sprint 中新增需求 | 团队直接接受保持客户满意 | PO 纳入 Product Backlog，协商对 Sprint 影响 | Follow Process（Agile Backlog） |
| 变更已批准 | 继续按旧计划执行 | 更新项目管理计划与基准后执行 | Update plan before execution |

### 风险

| 场景 | 错误动作（看似合理） | 优先动作 | PMP 原则 |
|------|----------------------|----------|----------|
| 识别到新风险 | 立即上报发起人 | 更新风险登记册，定性/定量分析 | Analysis Before Action |
| 风险已发生 | 仅记入风险登记册 | 登记问题日志，采取纠正措施 | Root Cause + 区分 Risk/Issue |
| 高概率威胁 | 直接规避改架构（未评估） | 分析影响 → 规划应对 → 实施 | Analysis + Follow Process |
| 低风险 | 花费大量资源消除 | 主动或被动接受（视影响） | Most Appropriate（不过度反应） |
| 重复出现的风险 | 每次临时救火 | 根因分析 + 预防措施 | Root Cause + Preventive Action |

### 干系人冲突

| 场景 | 错误动作（看似合理） | 优先动作 | PMP 原则 |
|------|----------------------|----------|----------|
| 关键相关方抵制 | 忽视或强制推进 | 沟通了解顾虑，调整参与策略 | Collaborate Before Escalate |
| 期望与交付不符 | 口头承诺新范围 | 管理期望，澄清章程/计划与变更流程 | Follow Process + Collaborate |
| 多方利益冲突 | 立即请发起人裁决 | 促进相关方协商，寻求折中方案 | Collaborate Before Escalate |
| 沟通不畅 | 发长篇邮件了事 | 选择合适沟通方式（常面对面）面对面沟通 | Collaborate |
| 相关方未识别全 | 直接制定计划 | 更新相关方登记册，再制定参与策略 | Analysis Before Action |

### 质量问题

| 场景 | 错误动作（看似合理） | 优先动作 | PMP 原则 |
|------|----------------------|----------|----------|
| 缺陷反复出现 | 加大测试力度收尾 | 根因分析，改进过程 | Root Cause Before Solution |
| 可交付成果不符标准 | 先交付后修补 | 按质量控制流程处理，不符合则不验收 | Follow Process |
| 质量审计发现偏差 | 处罚责任人 | 分析过程偏差，实施纠正与预防 | Root Cause + Preventive Action |
| 客户验收争议 | PM 单方面让步 | 依据验收标准与合同协商 | Collaborate + Follow Process |
| Sprint Increment 未达 DoD | 演示时解释说明 | 不视为完成；回到开发或调整范围 | Follow Process（Agile DoD） |

### 团队冲突

| 场景 | 错误动作（看似合理） | 优先动作 | PMP 原则 |
|------|----------------------|----------|----------|
| 两名成员争执 | 上报发起人或换人 | 面对面沟通，合作/妥协解决问题 | Collaborate Before Escalate |
| 绩效不佳 | PM 单独决定处罚 | 了解原因，教练、培训或资源支持 | Analysis + Team Participation |
| 虚拟团队误解 | 增加邮件频率 | 视频会议等高带宽沟通 | Collaborate |
| 技能缺口 | PM 外聘不告知团队 | 与团队讨论方案（培训、调配、支持） | Team Participation |
| 敏捷团队分歧 | SM 替团队做决定 | 促进团队自组织讨论达成共识 | Team Participation |

### 资源问题

| 场景 | 错误动作（看似合理） | 优先动作 | PMP 原则 |
|------|----------------------|----------|----------|
| 关键成员缺席 | 立即加班由他人硬顶 | 评估影响 → 资源平衡/获取 → 更新计划 | Analysis Before Action |
| 矩阵环境资源争用 | PM 越权指挥职能经理的人 | 与职能经理协商资源 | Collaborate Before Escalate |
| 资源不足 | 削减范围不沟通 | 分析备选方案，走变更或优先级排序 | Analysis + Follow Process |
| 多项目抢人 | 直接找高管要人 | 先项目间协商、资源平衡，再升级 | Collaborate Before Escalate |
| 团队超负荷 | 要求持续加班赶工 | 重新评估优先级、范围或进度（变更/Backlog） | Analysis + Sustainable pace |

---

## Exam Trap

### 为什么很多选项看似正确，但不是最佳答案

PMP 考试不考「能不能想到一个合理动作」，而考「在 PMI 价值观与流程下，**当前情境的最优下一步**是什么」。干扰项常用以下套路：

| 陷阱类型 | 看似正确的原因 | 为何不是最佳 | 优先级原则 |
|----------|----------------|--------------|------------|
| **过早行动** | 果断、高效、客户导向 | 跳过分析与记录 | Analysis Before Action |
| **过早升级** | 体现重视、推责清晰 | 未尝试团队/相关方协作 | Collaborate Before Escalate |
| **绕过流程** | 节省时间、灵活应变 | 破坏基准与变更控制 | Follow Process Before Changing |
| **治标不治本** | 快速恢复交付 | 未防再发 | Root Cause / Preventive Action |
| **PM 英雄主义** | 展现领导力 | 剥夺团队参与与自组织 | Team Participation |
| **过度反应** | 显得重视风险/质量 | 成本过高、不符合 Most Appropriate | 情境适配 |
| **反应不足** | 显得低调谨慎 | 忽视问题登记与正式管理 | Analysis + Follow Process |
| **First 题选 Best** | 选项是「最终好结果」 | 问的是第一步不是终局 | 流程顺序 |
| **混淆角色** | 动作本身合理 | 不该由该角色执行（PO/SM/PM） | 角色边界 |
| **混淆 Risk/Issue** | 都在「处理问题」 | 已发生应用问题日志 | Analysis |
| **绝对化措辞** | 语气肯定 | always / never / immediately 多为错 | 排除极端项 |
| **文档过度** | 显得专业规范 | 敏捷情境应面对面轻文档 | 情境适配 |
| **文档不足** | 敏捷拥抱变化 | 预测型情境仍需书面变更记录 | Follow Process |

### Agent 仲裁流程（多选项平局）

```
1. 确认问法：First / Next / Best / Most Appropriate
2. 用 decision_tree 缩窄到 2+ 合理选项
3. 对每个选项标注命中的 Core Principle（可多条）
4. 选「最高优先级原则」匹配最深的选项
5. 若仍平局：
     - First  → 选流程最靠前（分析/记录 > 执行/升级）
     - Best   → 选覆盖协作+流程+预防最完整者
     - Most Appropriate → 选反应适度、无越权、无过度升级者
6. 在输出中写明：「B 优于 C，因 C 违反 Collaborate Before Escalate」
```

### 输出示例

```markdown
**优先级仲裁**
- A：立即上报发起人 → 违反 P2 Collaborate Before Escalate
- B：与双方会面合作解决 → 符合 P1 + P2 ✓
- C：重新分配任务回避冲突 → 未解决根因，违反 P4
- D：更新资源管理计划 → 与当前冲突无直接关联
→ **最佳：B**（Collaborate Before Escalate + Analysis Before Action）
```
