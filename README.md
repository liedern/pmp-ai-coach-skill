# PMP AI Coach Skill

可维护的 Agent Skill 项目，用于打造 **PMP AI 教练**，帮助考生系统化备考与错题复盘。

## 目标

帮助用户：

1. 分析 PMP 错题
2. 自动识别知识点
3. 判断错误原因
4. 建立个人错题库
5. 制定学习计划
6. 支持未来接入 Cursor、OpenClaw、ChatGPT 等 Agent 平台

## 项目定位

本仓库不是传统应用代码库，而是一套可复用的 **Agent Skill**：

- 用结构化文档定义教练行为、工作流与数据模型
- 用知识库沉淀考试、PMBOK、敏捷相关内容
- 用模板统一输出格式，便于跨平台 Agent 调用

## 目录结构

```text
/
├── README.md                 # 项目说明
├── skill.md                  # Skill 定义与能力边界
├── version.md                # 版本信息
├── changelog.md              # 变更记录
│
├── knowledge/                # 领域知识
│   ├── exam/                 # 考试规则与题型
│   ├── pmbok/                # PMBOK 知识体系
│   └── agile/                # 敏捷相关知识
│
├── workflows/                # 核心工作流
│   ├── question_analysis.md  # 题目分析
│   ├── mistake_classification.md  # 错题归因
│   ├── study_plan.md         # 学习计划
│   └── mock_exam.md          # 模拟考试
│
├── database/                 # 数据模型（文档化 Schema）
│   ├── question_schema.md
│   ├── mistake_schema.md
│   └── learning_schema.md
│
├── templates/                # 输出模板
│   ├── wrong_question.md
│   ├── daily_report.md
│   └── study_plan.md
│
└── examples/                 # 示例输入输出
```

## 设计原则

- **可维护**：知识、流程、模板分离，便于迭代
- **可移植**：优先平台无关的 Markdown / Schema，便于接入多 Agent 平台
- **可扩展**：先定义能力与数据结构，再逐步补齐知识与工作流细节

## 当前状态

项目已完成目录初始化。业务内容（Skill 定义、工作流、Schema、模板、知识库）待后续逐步填充。

## 后续计划

1. 完善 `skill.md`：定义教练角色、输入输出与约束
2. 填写工作流文档：错题分析 → 归因 → 入库 → 学习计划
3. 定义数据库 Schema：题目、错题、学习进度
4. 补齐输出模板与示例
5. 按平台适配（Cursor / OpenClaw / ChatGPT 等）编写接入说明
