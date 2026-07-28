# Agile Logic（敏捷决策逻辑）

> **用途**：敏捷 / Scrum 情境题的识别信号、角色边界、事件顺序与需求变更处理规则。供 Agent 在 `project_type = agile` 时调用。
>
> **调用时机**：Step 1 判定为 Agile，或 Hybrid 题中当前动作属于迭代交付侧（Sprint、Backlog、评审会）。

---

## 1. 敏捷项目识别方法

满足以下**任一强信号**即可倾向 Agile；2 项以上高置信：

| 信号类型 | 关键词 / 场景线索 |
|----------|-------------------|
| 框架 | Scrum、Kanban、XP、SAFe（若题干出现） |
| 角色 | Product Owner、Scrum Master、Development Team、自组织团队 |
| 事件 | Sprint、Sprint Planning、Daily Scrum、Sprint Review、Retrospective |
| 工件 | Product Backlog、Sprint Backlog、Increment、用户故事、史诗 |
| 交付节奏 | 迭代、增量、2 周发布、MVP、持续交付 |
| 实践 | 站会、结对编程、TDD、燃尽图、看板、WIP 限制 |
| 价值观 | 拥抱变更、客户协作、响应变化胜过遵循计划 |

**排除信号**（倾向 Predictive）：

- CCB 审批基准变更、详细 WBS 先于交付、阶段门、挣值基准控制（无迭代语境）

**Hybrid 处理**：题干同时有「发布里程碑」和「两周 Sprint」→ 判断题目焦点：

- 迭代内优先级/需求 → Agile 逻辑
- 跨阶段合同/合规门 → Predictive 逻辑

---

## 2. Scrum 核心角色

| 角色 | 核心职责 | 考试边界（不该做的事） |
|------|----------|------------------------|
| **Product Owner（PO）** | 最大化产品价值；管理 Product Backlog；排优先级；澄清需求 | 不微管团队怎么实现；不代替 SM 清障碍；不单方面改 Sprint Goal 已承诺范围（须与团队协商） |
| **Scrum Master（SM）** | 促进 Scrum 执行；教练团队；移除障碍；保护团队免受干扰；促进协作 | 不是项目经理式的任务分派者；不是产品决策者；不替 PO 排优先级 |
| **Development Team（开发团队）** | 自组织完成 Sprint Backlog；估算；交付可工作的 Increment | 不接受外部随意加活破坏 Sprint；跨职能集体负责 |

**无传统 PM 时**：整合、障碍清除、流程促进 → 常由 SM + 团队协作承担；**不要**在敏捷题中选「PM 直接分派任务给成员」。

**冲突升级路径**：

```
团队内部 → SM 促进 → PO 澄清价值/优先级 → 组织/管理层（仅当 Scrum 内无法解决）
```

---

## 3. Scrum 事件

| 事件 | 目的 | 时长参考 | 关键规则 |
|------|------|----------|----------|
| **Sprint** | 固定周期交付 Increment | 通常 ≤ 1 月 | Sprint 期间目标稳定；范围变更不随意插入 |
| **Sprint Planning** | 确定 Sprint Goal + Sprint Backlog | 时间盒 | PO 带优先级条目；团队承诺能完成的工作 |
| **Daily Scrum** | 同步进度、暴露障碍 | 15 分钟 | 开发团队为主；谈昨天/今天/障碍，非状态汇报会 |
| **Sprint Review** | 检视 Increment、收集反馈 | 时间盒 | 相关方参与；反馈进入 Backlog，非当场承诺进当前 Sprint |
| **Sprint Retrospective** | 改进流程与协作 | 时间盒 | 团队内部；产出改进行动项 |

**事件顺序（时间线）**：

```
Sprint Planning → Daily Scrum（每日）→ Sprint Review → Retrospective → 下一 Sprint
```

| 问法 | 倾向答案 |
|------|----------|
| 相关方想看成果 | Sprint Review（非 Retrospective） |
| 团队改进工作方式 | Retrospective |
| 发现障碍 | Daily Scrum 提出 → SM 跟进移除 |
| 确定本迭代做什么 | Sprint Planning |

---

## 4. Product Backlog 逻辑

```
Product Backlog（PO 拥有，按价值排序）
        │
        ▼ Sprint Planning（团队与 PO 协商）
Sprint Backlog（团队拥有，本 Sprint 承诺）
        │
        ▼ 开发 + Daily Scrum
Increment（符合 DoD 的可工作产品）
        │
        ▼ Sprint Review 反馈
回到 Product Backlog 重排优先级
```

| 规则 | 说明 |
|------|------|
| 唯一来源 | 所有需求变更先进入 Product Backlog，由 PO 排序 |
| 细化（Refinement） | 持续进行，通常消耗 Backlog 10% 容量；**不是**单独强制仪式名称考点时答 Planning |
| 优先级 | 价值、风险、依赖、学习；PO 决策，团队可协商 |
| Definition of Ready | 条目进入 Sprint 前应够清晰（考点：未就绪不应强行承诺） |
| Definition of Done（DoD） | Increment 质量标准；未达 DoD 不算完成 |

**考试指向**：

- 新需求 / 变更想法 → PO 更新 Backlog，**不是**直接塞进当前 Sprint
- 优先级冲突 → PO 与相关方/团队澄清价值后重排

---

## 5. 需求变化处理方式

Sprint **进行中**收到变更：

```
1. 记录需求（PO 纳入 Product Backlog）
2. 评估对 Sprint Goal 的影响
3. 与 PO、团队协作：
   - 若可吸收且不伤 Sprint Goal → 协商调整 Sprint Backlog
   - 若严重影响 Sprint Goal → 取消 Sprint 或等下一 Sprint（极端）
4. 不在 Sprint 内静默接受大量范围膨胀
```

| 情境 | 优先动作 |
|------|----------|
| 客户演示后新想法 | Review 反馈 → Backlog → PO 排序 → **下一 Sprint** 考虑 |
| 紧急缺陷 | 团队与 PO 协商；可能用 Backlog 替换等量工作 |
| 法规突变 | PO 重排 Backlog；与团队重协商 Sprint 目标 |
| 高管插队 | PO 负责说「不」或换优先级，SM 保护团队专注 |

**敏捷变更哲学**：拥抱变更，但通过 **Backlog 与优先级透明化** 管理，而非无流程地随时加活。

**Hybrid**：对外合同里程碑不变 + 内部用 Sprint → 外部变更走合同/变更流程，内部用 Backlog 重排。

---

## 6. Agile PM 行为原则

敏捷语境下 PM 或 SM/PO 类角色应遵循：

```
A1  Value first           → 优先交付最大价值（PO 排序依据）
A2  Empower the team      → 自组织、不微管
A3  Transparent inspect   → 看板/燃尽图/评审会暴露问题
A4  Adapt over contract   → 响应变化，通过 Backlog 调整
A5  Collaborate with customer → 客户/相关方持续参与 Review
A6  Remove impediments    → 障碍优先由 SM 清除
A7  Sustainable pace      → 不鼓励长期加班换速度
A8  Face-to-face talk     → 高带宽沟通优于冗长文档（非「零文档」）
```

**与 Predictive 原则对照**：

| Predictive | Agile |
|------------|-------|
| 变更 → CCB | 变更 → Backlog + PO |
| 计划基准 | Sprint Goal + Increment |
| 上报发起人 | SM 促进、PO 决策、团队自解 |
| 详细前期计划 | 滚动式规划、Just-in-time |

**Servant Leadership（仆人式领导）**：服务团队、清障碍、教练 — 考试选「支持团队」而非「命令控制」。

---

## 7. 常见考试陷阱（Agile）

| 陷阱 | 错误倾向 | 正确倾向 |
|------|----------|----------|
| PM 命令式 | PM 分配任务、逐人 micromanage | 自组织；SM 教练 |
| 角色混淆 | SM 定优先级 / PO 清技术障碍 | PO=价值；SM=流程与障碍 |
| Sprint 中加需求 | 直接接受所有新需求 | Backlog；协商替换或下一 Sprint |
| Daily Scrum 误用 | 向 PM 汇报、长篇讨论 | 15 分钟同步；细节另开会 |
| Review vs Retro | 向相关方做改进会 | Review=展示成果；Retro=团队改进 |
| 无 Increment | Sprint 结束仅文档 | 可工作、符合 DoD 的产品增量 |
| 跳过 PO | 团队自行决定产品优先级 | PO 对 Backlog 负责 |
| 过度文档 | 补全厚重计划再开发 | 刚好够用的信息 + 面对面 |
| 冲突即上报 | 第一件事找高管 | SM 促进、团队协作解决 |
| Hybrid 误用 | 迭代内走 CCB 改 Sprint Goal | 分清外部治理与内部迭代 |
| 速度崇拜 | 牺牲质量赶工 | DoD、可持续节奏 |
| 零变更 | 「敏捷不接受变更」 | 通过 Backlog 管理变更 |

---

## 8. Agent 调用检查清单

分析 Agile 情境题时，依次确认：

- [ ] 强信号是否支持 Agile（角色/事件/工件）
- [ ] 涉及哪个角色（PO / SM / Team）— 选项是否越权
- [ ] 处于哪个事件或 Sprint 时间点
- [ ] 变更是进 Backlog 还是 Sprint 内协商
- [ ] 问法是 First / Next / Best
- [ ] 是否触犯 §7 陷阱
