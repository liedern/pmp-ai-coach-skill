# PMP Glossary（术语总表）

> **来源**：【事实】`source_materials/terminology/PMP中英文词组翻译.pdf`  
> **加工**：`workflows/material_processing.md` | `import_log.md`  
> **用途**：PMP 双语术语主索引；Agent 查词、Terminology Problem 错因分析。  
> **详解**：高频词条见 `knowledge/language/pmp_terms.md`；易混见 `confusing_terms.md`。

---

## 使用说明

| 列 | 说明 |
|----|------|
| English Term | 英文原词/缩写 |
| 中文名称 | 【事实】PDF 译文 |
| PMP考试含义 | 应试一句话（【事实】源自 PDF 注释或【推测】考试常用理解） |
| 关键词 | 题干信号词 |
| 易混概念 | 对比项 |

---

## 一、成本与挣值管理

| English Term | 中文名称 | PMP考试含义 | 关键词 | 易混概念 |
|--------------|----------|-------------|--------|----------|
| AC | 实际成本 | 截至某时点已花费的实际成本 | actual cost, spent | ACWP（旧称） |
| ACWP | 已完成工作实际成本 | 旧称，现通常用 AC | legacy | AC |
| BAC | 完工预算 | 项目总批准预算 | budget at completion | EAC |
| BCWP | 已完成工作预算成本 | 旧称，现通常用 EV | legacy | EV |
| BCWS | 计划工作预算成本 | 旧称，现通常用 PV | legacy | PV |
| CPI | 成本绩效指数 | EV÷AC，成本效率 | cost performance | SPI |
| CV | 成本偏差 | EV−AC | cost variance | SV |
| EAC | 完工估算 | 项目完成时总成本预测 | estimate at completion | ETC, BAC |
| EACt | 预计完工时间 | 非 PMI 核心统一缩写 | time at completion | EAC |
| ETC | 完工尚需估算 | 完成剩余工作所需成本 | estimate to complete | EAC |
| EV | 挣值 | 已完成工作的预算价值 | earned value | AC, PV |
| EVA | 挣值分析 | 用挣值数据分析绩效 | earned value analysis | EVM |
| EVM | 挣值管理 | 整合范围/进度/成本绩效 | earned value management | — |
| PMB | 绩效测量基准 | 范围+进度+成本基准组合 | performance measurement baseline | baseline |
| PV | 计划价值 | 计划完成工作的预算价值（EVM） | planned value | Present Value（财务） |
| SPI | 进度绩效指数 | EV÷PV，进度效率 | schedule performance | CPI |
| SV | 进度偏差 | EV−PV | schedule variance | CV |
| TCPI | 完工尚需绩效指数 | 剩余工作需达到的效率 | to-complete performance index | CPI |
| VAC | 完工偏差 | BAC−EAC | variance at completion | CV, SV |

---

## 二、挣值常用公式

| 公式 | 中文 | PMP考试含义 | 关键词 |
|------|------|-------------|--------|
| CV = EV − AC | 成本偏差 | CV<0 超支 | cost variance |
| SV = EV − PV | 进度偏差 | SV<0 落后 | schedule variance |
| CPI = EV ÷ AC | 成本绩效指数 | <1 超支 | CPI |
| SPI = EV ÷ PV | 进度绩效指数 | <1 落后 | SPI |
| ETC = EAC − AC | 完工尚需估算 | 还需花多少 | ETC |
| VAC = BAC − EAC | 完工偏差 | 完工时超/节支多少 | VAC |
| EAC = BAC ÷ CPI | 按当前成本绩效预测完工成本 | 典型 EAC 公式 | EAC |
| EAC = AC + BAC − EV | 后续按预算效率执行 | 非典型偏差 | EAC |
| EAC = BAC ÷ (CPI×SPI) | 成本进度绩效均持续 | 综合偏差 | EAC |
| EAC = AC + Bottom-up ETC | 原估算失效重新估算 | 自下而上 | EAC |
| TCPI = (BAC−EV)/(BAC−AC) | 以 BAC 为目标的 TCPI | 剩余需达效率 | TCPI |
| TCPI = (BAC−EV)/(EAC−AC) | 以 EAC 为目标的 TCPI | 修订目标 | TCPI |

---

## 三、进度与时间管理

| English Term | 中文名称 | PMP考试含义 | 关键词 | 易混概念 |
|--------------|----------|-------------|--------|----------|
| CP | 关键路径 | 最长路径决定最短工期 | critical path | critical chain |
| CPA | 关键路径活动 | 位于关键路径上的活动 | critical path activity | — |
| CPM | 关键路径法 | 计算关键路径的方法 | critical path method | PERT |
| EF | 最早完成时间 | 网络图时间参数 | early finish | LF |
| ES | 最早开始时间 | 网络图时间参数 | early start | LS |
| FF | 自由浮动时间 | 不影响后继最早开始 | free float | Finish-to-Finish |
| LF | 最晚完成时间 | 网络图时间参数 | late finish | EF |
| LS | 最晚开始时间 | 网络图时间参数 | late start | ES |
| TF | 总浮动时间 | 不影响项目完工的延迟量 | total float | free float |
| Lead | 提前量 | 加速后继开始 | lead time | lag |
| Lag | 滞后量 | 推迟后继开始 | lag time | lead |
| Milestone | 里程碑 | 零持续时间的重大节点 | milestone | deliverable |
| Schedule Baseline | 进度基准 | 批准的计划进度 | baseline | schedule |
| Schedule Compression | 进度压缩 | 缩短工期不缩范围 | compression | — |
| Fast Tracking | 快速跟进 | 并行原本顺序活动 | parallel | crashing |
| Crashing | 赶工 | 加资源缩短工期 | crash, resources | fast tracking |
| Resource Leveling | 资源平衡 | 可能改变关键路径/工期 | leveling | smoothing |
| Resource Smoothing | 资源平滑 | 一般不改关键路径 | smoothing | leveling |
| Rolling Wave Planning | 滚动式规划 | 近期细、远期粗 | rolling wave | — |
| Progressive Elaboration | 渐进明细 | 随信息增加细化计划 | progressive elaboration | — |
| WIP | 进行中的工作 | 在制品（敏捷/精益） | work in progress | — |

---

## 四、活动逻辑关系

| English Term | 中文名称 | PMP考试含义 | 关键词 |
|--------------|----------|-------------|--------|
| FS | 完成到开始 | 最常见依赖关系 | finish-to-start |
| FF | 完成到完成 | 两活动同时完成 | finish-to-finish |
| SS | 开始到开始 | 同时开始 | start-to-start |
| SF | 开始到完成 | 少见依赖 | start-to-finish |

---

## 五、网络图与估算技术

| English Term | 中文名称 | PMP考试含义 | 关键词 | 易混概念 |
|--------------|----------|-------------|--------|----------|
| ADM | 箭线图法 | 旧法，箭线表活动 | arrow diagramming | PDM |
| AOA | 箭线表示活动 | 同 ADM | activity on arrow | AON |
| AON | 节点表示活动 | 现代标准 | activity on node | AOA |
| PDM | 紧前关系绘图法 | 创建网络图主流方法 | precedence diagramming | ADM |
| GERT | 图形评审技术 | 允许回路条件分支 | GERT | PDM |
| PERT | 计划评审技术 | 三点估算处理不确定性 | PERT | CPM |
| LOE | 支持型活动 | 无明确可交付成果的持续工作 | level of effort | — |
| Three-Point Estimating | 三点估算 | 乐观+最可能+悲观 | three-point | analogous |
| Analogous Estimating | 类比估算 | 参考历史项目，快但粗 | analogous, top-down | parametric |
| Parametric Estimating | 参数估算 | 统计关系计算 | parametric | analogous |
| Bottom-up Estimating | 自下而上估算 | 最准最耗时 | bottom-up | analogous |
| Expert Judgment | 专家判断 | 咨询专家 | expert judgment | — |
| Monte Carlo Simulation | 蒙特卡洛模拟 | 概率风险模拟 | monte carlo | — |
| Critical Chain Method | 关键链法 | 管理缓冲与资源约束 | critical chain | CPM |
| Project Buffer | 项目缓冲 | 关键链末端保护工期 | buffer | feeding buffer |
| Feeding Buffer | 接驳缓冲 | 非关键链汇入点缓冲 | feeding buffer | — |

---

## 六、范围管理

| English Term | 中文名称 | PMP考试含义 | 关键词 | 易混概念 |
|--------------|----------|-------------|--------|----------|
| WBS | 工作分解结构 | 可交付成果层级分解 | work breakdown structure | PBS |
| WBS Dictionary | WBS 词典 | WBS 组件详细说明 | dictionary | — |
| Work Package | 工作包 | WBS 最低可管理单元 | work package | planning package |
| Planning Package | 规划包 | 已知内容但未详细规划 | planning package | work package |
| Control Account | 控制账户 | 整合范围/进度/成本的管理点 | control account | — |
| Scope Baseline | 范围基准 | 批准的范围+WBS+词典 | scope baseline | — |
| Product Scope | 产品范围 | 产品/服务特性与功能 | product scope | project scope |
| Project Scope | 项目范围 | 为交付产品所需工作 | project scope | product scope |
| Scope Creep | 范围蔓延 | 未批准范围扩大 | scope creep | gold plating |
| Gold Plating | 镀金 | 擅自增加未批准功能 | gold plating | scope creep |
| Requirements Traceability Matrix | 需求跟踪矩阵 | 需求↔可交付成果↔测试 | RTM | — |
| Acceptance Criteria | 验收标准 | 可交付成果验收条件 | acceptance criteria | DoD |
| Deliverable | 可交付成果 | 可验证的输出 | deliverable | — |
| Validate Scope | 确认范围 | 正式验收可交付成果 | customer acceptance | Control Quality |
| Control Scope | 控制范围 | 监督范围、管变更 | control scope | Validate Scope |
| Decomposition | 分解 | 拆分 WBS | decomposition | — |
| Product Analysis | 产品分析 | 定义产品方案 | product analysis | — |
| PBS | 产品分解结构 | 按产品组件分解 | product breakdown structure | WBS |

---

## 七、组织结构与责任分配

| English Term | 中文名称 | PMP考试含义 | 关键词 | 易混概念 |
|--------------|----------|-------------|--------|----------|
| OBS | 组织分解结构 | 按组织单元分解 | organizational breakdown | WBS |
| RAM | 责任分配矩阵 | 角色与职责映射总称 | responsibility assignment | RACI |
| RACI | 执行/负责/征询/知情 | RAM 具体形式 | responsible, accountable | RAM |
| R | Responsible | 实际执行者 | do the work | Accountable |
| A | Accountable | 对结果负责并批准者 | approve | Responsible |
| C | Consulted | 需征询意见者 | consult | Informed |
| I | Informed | 需被告知者 | inform | Consulted |
| RBS | 风险分解结构 | 按风险类别分解 | risk breakdown | Resource RBS |
| Resource Breakdown Structure | 资源分解结构 | 按资源类型分解 | resource breakdown | Risk RBS |
| Functional Organization | 职能型组织 | PM 权力弱 | functional | projectized |
| Matrix Organization | 矩阵型组织 | 双重汇报 | matrix | — |
| Weak/Balanced/Strong Matrix | 弱/平衡/强矩阵 | PM 权力递增 | matrix type | — |
| Projectized Organization | 项目型组织 | PM 权力强 | projectized | functional |
| Composite Organization | 复合型组织 | 混合结构 | composite | — |

---

## 八、整合与变更管理

| English Term | 中文名称 | PMP考试含义 | 关键词 | 易混概念 |
|--------------|----------|-------------|--------|----------|
| CCB | 变更控制委员会 | 审查批准变更 | change control board | — |
| CCS | 变更控制系统 | 变更管理程序 | change control system | — |
| CR | 变更请求 | 正式修改提议 | change request | direct change |
| Integrated Change Control | 实施整体变更控制 | 审查并管理基准变更 | integrated change control | execute |
| Corrective Action | 纠正措施 | 纠正已发生偏差 | corrective | preventive |
| Preventive Action | 预防措施 | 防止未来偏差 | preventive | corrective |
| Defect Repair | 缺陷补救 | 修复不符合项 | defect repair | — |
| Configuration Management | 配置管理 | 管理产品/文档版本 | configuration | — |
| Change Log | 变更日志 | 记录所有变更 | change log | — |
| Project Charter | 项目章程 | 授权项目，发起人批准 | charter | plan |
| Project Management Plan | 项目管理计划 | PM 制定的整合计划 | PM plan | charter |
| Benefits Management Plan | 效益管理计划 | 如何创造效益 | benefits | — |
| Business Case | 商业论证 | 项目商业价值依据 | business case | — |
| Assumption Log | 假设日志 | 记录假设 | assumption log | — |
| Issue Log | 问题日志 | 已发生事项 | issue log | risk register |
| Lessons Learned Register | 经验教训登记册 | 项目内经验教训 | lessons learned | repository |
| Lessons Learned Repository | 经验教训知识库 | 组织级 OPA | repository | register |
| Work Performance Data | 工作绩效数据 | 原始观察数据 | raw data | information |
| Work Performance Information | 工作绩效信息 | 分析后的绩效信息 | analyzed | data, reports |
| Work Performance Reports | 工作绩效报告 | 格式化报告供决策 | reports | information |

---

## 九、组织环境与项目治理

| English Term | 中文名称 | PMP考试含义 | 关键词 | 易混概念 |
|--------------|----------|-------------|--------|----------|
| EEF | 事业环境因素 | 组织外部/内部约束条件 | enterprise environmental | OPA |
| OPA | 组织过程资产 | 流程、模板、历史信息 | organizational process assets | EEF |
| PMBOK | 项目管理知识体系 | PMI 知识体系 | PMBOK | PMB |
| PMI | 项目管理协会 | 发证机构 | PMI | — |
| PMIS | 项目管理信息系统 | 项目管理工具系统 | PMIS | — |
| PMO | 项目管理办公室 | 项目治理与支持机构 | PMO | PMB |
| VDO | 价值交付办公室 | 价值导向治理机构 | value delivery office | PMO |
| Portfolio | 项目组合 | 战略目标导向的项目集合 | portfolio | program |
| Program | 项目集 | 相关项目协同获益 | program | portfolio |
| Supportive/Controlling/Directive PMO | 支持/控制/指令型 PMO | PMO 权力递增 | PMO type | — |

---

## 十、风险管理

| English Term | 中文名称 | PMP考试含义 | 关键词 | 易混概念 |
|--------------|----------|-------------|--------|----------|
| Risk | 风险 | 尚未发生的不确定事件 | uncertain, future | Issue |
| Issue | 问题 | 已发生需处理的事项 | occurred, problem | Risk |
| Risk Register | 风险登记册 | 已识别风险及应对 | risk register | issue log |
| Risk Report | 风险报告 | 向相关方报告风险信息 | risk report | — |
| Contingency Reserve | 应急储备 | 已知未知风险储备 | contingency | management reserve |
| Management Reserve | 管理储备 | 未知未知，需批准使用 | management reserve | contingency |
| EMV | 预期货币价值 | 概率×影响货币化 | expected monetary value | — |
| Workaround | 权变措施 | 未计划风险的临时应对 | workaround | contingency plan |
| Residual Risk | 残余风险 | 应对后剩余风险 | residual | secondary |
| Secondary Risk | 次生风险 | 应对行动产生的新风险 | secondary | residual |

### 威胁应对 / 机会应对

| 威胁策略 | 中文 | 机会策略 | 中文 |
|----------|------|----------|------|
| Avoid | 规避 | Exploit | 开拓/利用 |
| Mitigate | 减轻 | Enhance | 提高 |
| Transfer | 转移 | Share | 分享 |
| Accept | 接受 | Accept | 接受 |
| Escalate | 上报 | Escalate | 上报 |

---

## 十一～十三、质量管理与工具

| English Term | 中文名称 | PMP考试含义 | 易混概念 |
|--------------|----------|-------------|----------|
| Manage Quality | 管理质量 | 过程质量保证/审计 | Control Quality |
| Control Quality | 控制质量 | 检查可交付成果 | Validate Scope |
| Quality Audit | 质量审计 | 过程是否符合计划 | inspection |
| Cause-and-Effect Diagram | 因果图/鱼骨图 | 根因分析工具 | — |
| Pareto Chart | 帕累托图 | 80/20 问题排序 | — |
| Control Chart | 控制图 | 过程稳定性 | specification limit |

---

## 十四、采购与合同

| English Term | 中文名称 | PMP考试含义 | 易混概念 |
|--------------|----------|-------------|----------|
| FFP | 固定总价合同 | 卖方成本风险高 | CPFF |
| FPIF | 总价加激励费用合同 | 固定价+激励 | FFP |
| CPFF/CPIF/CPAF | 成本加费用合同族 | 买方成本风险高 | FFP |
| T&M | 工料合同 | 范围不确定短期 | FFP |
| SOW | 工作说明书 | 采购范围描述 | procurement SOW |
| RFP/RFQ/RFI | 建议/报价/信息邀请书 | 不同阶段采购文件 | — |
| Make-or-Buy Analysis | 自制或外购分析 | 决定自制还是外购 | — |

---

## 十五、财务与商业评估

| English Term | 中文名称 | PMP考试含义 | 易混概念 |
|--------------|----------|-------------|----------|
| NPV | 净现值 | 越大越优先（通常） | ROI |
| ROI | 投资回报率 | 投资回报比例 | NPV |
| IRR | 内部收益率 | 折现率为零的收益率 | — |
| Sunk Cost | 沉没成本 | 不应影响未来决策 | opportunity cost |
| Opportunity Cost | 机会成本 | 放弃的最佳替代 | sunk cost |
| Payback Period | 投资回收期 | 收回投资所需时间 | — |

---

## 十六、沟通与干系人

| English Term | 中文名称 | PMP考试含义 | 关键词 |
|--------------|----------|-------------|--------|
| Stakeholder Register | 相关方登记册 | 相关方信息与分析 | stakeholder |
| Power-Interest Grid | 权力利益方格 | 相关方分析工具 | power, interest |
| Salience Model | 凸显模型 | 权力/紧迫/合法性 | salience |
| Interactive/Push/Pull Communication | 互动/推送/拉式沟通 | 不同沟通技术 | communication |
| Conflict Management | 冲突管理 | 处理团队/相关方冲突 | conflict |

---

## 十七～十八、团队与冲突

| English Term | 中文名称 | PMP考试含义 | 易混概念 |
|--------------|----------|-------------|----------|
| Collaborate/Problem Solve | 合作解决问题 | 冲突策略首选（长期） | Compromise |
| Compromise/Reconcile | 妥协调解 | 双方让步 | Collaborate |
| Smooth/Accommodate | 缓和包容 | 强调一致淡化分歧 | Force |
| Force/Direct | 强迫命令 | 紧急或输赢场景 | Collaborate |
| Withdraw/Avoid | 撤退回避 | 暂避冲突 | Collaborate |
| Forming/Storming/Norming/Performing/Adjourning | 塔克曼五阶段 | 团队发展阶段 | — |
| Team Charter | 团队章程 | 团队价值观与规则 | project charter |

---

## 十九～二十四、敏捷 / Scrum / 精益

| English Term | 中文名称 | PMP考试含义 | 易混概念 |
|--------------|----------|-------------|----------|
| Product Owner | 产品负责人 | 价值与 Backlog 优先级 | Scrum Master |
| Scrum Master | Scrum Master | 流程/障碍/教练 | Product Owner |
| Sprint | 冲刺/迭代 | 固定周期交付增量 | iteration |
| Product Backlog | 产品待办列表 | PO 排序的需求池 | Sprint Backlog |
| Sprint Backlog | 迭代待办列表 | 本 Sprint 承诺工作 | Product Backlog |
| Definition of Done | 完成的定义 | 增量质量标准 | DoR |
| Definition of Ready | 准备就绪的定义 | 非官方 Scrum 工件，常用 | DoD |
| Velocity | 速率 | 每迭代完成故事点 | — |
| Burndown/Burnup Chart | 燃尽/燃起图 | 进度可视化 | — |
| WIP Limit | 在制品限制 | 限制并行工作 | — |
| Technical Debt | 技术债务 | 短期妥协累积 | — |
| Predictive/Adaptive/Hybrid Life Cycle | 预测/适应/混合生命周期 | 项目方法选择 | — |

---

## 二十五～二十六、商业价值与伦理

| English Term | 中文名称 | PMP考试含义 |
|--------------|----------|-------------|
| Value Proposition | 价值主张 | 产品/项目价值承诺 |
| Benefits Realization | 效益实现 | 交付并验证效益 |
| VOC | 客户之声 | 收集客户需求 |
| Responsibility/Respect/Fairness/Honesty | 责任/尊重/公平/诚实 | PMI 道德四大价值观 |

---

## 三十四～三十六、易混淆缩写（重点）

| English Term | 中文/说明 | 易混提示 |
|--------------|-----------|----------|
| PV | 计划价值（EVM）vs 现值（财务） | 看语境：挣值 vs NPV |
| FF | 自由浮动时间 vs 完成到完成关系 | 看语境：float vs dependency |
| RBS | 风险分解结构 vs 资源分解结构 | 看章节/上下文 |
| AC/EV/PV | 现行 EVM 术语 | vs ACWP/BCWP/BCWS 旧称 |
| SOW/RAM/OBS/PMB/PMBOK/PMO | 见第七章/第九章 | 字母相近易混 |
| FOW, PF | 【Need Review】源 PDF 标注非标准 | 使用 TF/FF |

---

## 扩展

- 详解除 `knowledge/language/pmp_terms.md`
- 易混对比见 `knowledge/language/confusing_terms.md`
- 同义词/旧称见 `knowledge/language/synonym_mapping.md`
