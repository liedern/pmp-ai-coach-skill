# 示例：学习复盘报告（Review Coach 输出摘要）

> 触发：用户说 **「复盘」**  
> 工作流：[`workflows/review_retrospective.md`](../workflows/review_retrospective.md)  
> 输入源：**现场读取** `mistake_memory.json`、`question_history.json`、`weak_points.json`（**不**读 `review_retrospective.json` 作聚合输入）  
> 写后可选更新：`memory/data/review_retrospective.json`（`review_snapshot` + `source_revisions`）

以下为 **Example Dataset** 虚构口径，演示 v0.1.0 规则，不对应任何真实用户 Memory。

---

# 学习复盘

> 数据范围：`mistake_memory` **2** 条；**确认错题 1** 条计入错因；**1** 条在 `question_history` 标 `answer_disputed`（展示用，错因占比不计入）。

## 1. 高频错误知识领域排序

| 排名 | 知识领域 | 说明 |
|------|----------|------|
| 1 | 挣值管理 / SPI | 确认错题，`concept_confusion` |
| 2 | 相关方管理 / 需求文件 | 关联 History 争议（`answer_disputed`），**不进**错因 share |

## 2. 高频错误类型分析

| 错误类型 | 次数 | 占比 |
|----------|------|------|
| `concept_confusion` | 1 | 100%（仅 **确认错题** 池） |

争议题、Bookmark **不参与**本节 `share` 汇总（v0.1.0 P0）。

## 3. 当前薄弱点

- **挣值管理 / 进度落后应对**（`weak_points.json`，`dominant_error_type: concept_confusion`）  
- 建议：SPI=EV/PV；落后时优先 Fast track / Crashing，勿与「引入 float」混淆。

## 4. 学习建议

1. 重做 `mistake_example_evm_001`（闭卷 + 口述 EV/PV 含义）。  
2. 争议关联题 `q_example_stakeholder_first_001`：按**动作**核对（团队+需求文件 vs 问题日志），不按选项字母死记。  
3. 若有收藏题，仅作考前自测，**不计入**错因排名。

## 5. 推荐下一步训练方向

**主推**：`review_mistake` — 确认错题重做 + EVM 进度压缩小专项（约 25 分钟）。

---

## 复盘后快照（可选持久化）

写入 `review_retrospective.json` 时须包含：

```json
{
  "record_kind": "review_snapshot",
  "snapshot_status": "current",
  "source_revisions": {
    "mistake_memory_updated_at": "<与 mistake_memory.updated_at 一致>",
    "question_history_updated_at": "<与 question_history.updated_at 一致>"
  },
  "last_retrospective": { }
}
```

机器可读完整结构见：`modules/review_coach/examples/sample_retrospective_output.json`。
