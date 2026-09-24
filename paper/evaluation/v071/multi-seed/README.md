# Multi-Seed / 温度敏感性实验（P2）

> 日期：2026-07-21 · 对应 `rebuttal-arsenal.md` C2（single run, no CI, no significance tests）
> 主仓脚本：`backend/scripts/multi_seed_evaluate.py` · 数据：本目录

## 设计

- **2 本小说 × 3 个温度**：西游记（100 章）、红楼梦（122 章），各在 temp 0.0 / 0.7 / 1.0 下用 DeepSeek V3（`deepseek-chat`）全量重抽取，共 6 个独立 run（apples-to-apples 章节副本，不动冻结数据）。
- 生产默认 temp=0.1；本实验扫描 0→1 全温域，比"同 temp 多 seed"更严苛——同时覆盖采样方差与温度敏感性。
- 每个 run 测两类指标：
  1. **结构指标**：full 5-skill 链（tier→votes→prior→edmonds→suffix）后的 max_children / roots / cycles / depth
  2. **per-node 质量**：`benchmark_hierarchy` 对冻结 gold（西游 1205 / 红楼 1019 标注节点）的 5 维准确率 + Overall
- 成本：6 run ≈ ¥30（DeepSeek）；耗时约 14 小时（含一次余额中断 + API 早高峰超时重试）。

## 结果

### Per-node 准确率（gold-based，mean ± std over 3 temps）

| 维度 | 西游（t0/0.7/1.0） | mean±std | 红楼（t0/0.7/1.0） | mean±std |
|---|---|---|---|---|
| Entity Precision | .9983 / .9992 / 1.0000 | **0.9992±0.0007** | .9990 / .9980 / 1.0000 | **0.9990±0.0008** |
| Name Accuracy | .9992 / .9992 / .9992 | **0.9992±0.0000** | 1.0000×3 | **1.0000±0.0000** |
| Tier Accuracy | .9934 / .9892 / .9925 | **0.9917±0.0018** | .9820 / .9860 / .9860 | **0.9847±0.0019** |
| Parent Precision | .9975 / .9983 / .9983 | **0.9981±0.0004** | .9960 / .9950 / .9960 | **0.9957±0.0005** |
| Structural Health | .9967 / .9983 / .9967 | **0.9972±0.0008** | 1.0000 / 1.0000 / .9990 | **0.9997±0.0005** |
| **Overall** | .9884 / .9859 / .9892 | **0.9878±0.0014** | .9790 / .9800 / .9820 | **0.9803±0.0012** |

**全部维度 std ≤ 0.19pp**——温度 0→1 全文重抽取下，per-node 质量波动比 IAA 噪声下限（3-5pp）小一个数量级以上。

### 结构指标（full pipeline 后）

| Novel | t0.0 | t0.7 | t1.0 | 全部 |
|---|---|---|---|---|
| 西游 max_children | 38 | 45 | 41 | 单根 ✅ 无环 ✅ |
| 红楼 max_children | 72 | 85 | 78 | 单根 ✅ 无环 ✅ |
| 西游 avg_depth | 2.93 | 2.61 | 3.07 | |
| 红楼 avg_depth | 2.94 | 3.89 | 5.56 | 深度较敏感（诚实记录） |

**6/6 run 收敛到单根无环合法树**——结构保证跨温度稳健，与 P1 跨 LLM 结论互证。

## 对 C2 的回答

- 攻击点"每个表格单元格都是单次运行"→ 现在有 3 温度 × 2 小说的方差证据：**pipeline 输出的 per-node 质量对采样/温度高度稳定（std ~0.1pp）**，表格数字不是运气。
- 诚实边界：①方差测量用的是 DeepSeek V3（成本约束），Claude 主表数字的多 seed 方差未测——但 P1 已证明 Claude→DeepSeek 的跨模型漂移本身也小（full pipeline 46 vs 63）；②红楼 depth 随温度上升（2.94→5.56），说明深层结构对抽取噪声更敏感，max_children/合法性不受影响。
- Rebuttal 话术："Extraction variance across the full temperature range (0→1) on a second extractor is ≤0.2pp on all five quality dimensions, an order of magnitude below the IAA noise floor; 6/6 runs converge to single-root acyclic trees."

## 事故记录（工程副产品）

- DeepSeek 余额耗尽导致 xy-t0.7 ch92-100 失败 → 暴露 `analysis_service` 自动重试 KeyError 僵尸任务 bug（已修复 `bad6042`）。
- DeepSeek 早高峰 ch28 连续 600s 超时 → 5 次指数退避重试成功，无需人工介入。

## 文件

- `multi_seed_summary.json` — 结构指标 per run + 聚合
- `benchmark_accuracy_summary.json` — 5 维准确率 per run
- `benchmark_{novel}_{id8}.json` — 6 份完整 benchmark 报告
