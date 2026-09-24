# Cross-LLM Replication: DeepSeek V3 抽取 × 西游记（P1 实验）

> 日期：2026-07-20 · 对应 `rebuttal-arsenal.md` C7（Single LLM, no cross-model validation）
> 主仓脚本：`backend/scripts/cross_llm_replication.py` · 原始结果：`result.json`

## 设计

- **对照原则 apples-to-apples**：新 novel（100 章）逐章复制自冻结的西游 v071 数据（is_excluded 一致），仅换抽取 LLM。
- **抽取器**：DeepSeek V3（`deepseek-chat`，经 OpenAI 兼容端点），100 章全量抽取，耗时 83 分钟，预估成本 $2.08 / ¥14.94（estimate 接口口径；未做 dashboard 对账）。
- **测量**：同一套 5-skill 链（tier → votes → prior → edmonds → suffix），分两档：
  - (a) 只跑 tier→votes：greedy voting 结构指标（对应论文 Table ablation 的 Voting 行，Claude 参照 279/25/2）
  - (b) 全链：full pipeline 结构指标（Claude 参照 63/1/0）
- KnowledgePrior 用书名"西游记"匹配，复用同一份 399 条先验表。

## 结果

| 指标 | Claude Sonnet 4（冻结） | DeepSeek V3 |
|---|---|---|
| **Voting: max_children** | **279** | **56** |
| Voting: roots | 25 | 5 |
| Voting: cycles | 2 | 0 |
| Voting: total_locations | ~640 | 537 |
| **Full: max_children** | **63** | **46** |
| Full: roots | 1 | 1 |
| Full: cycles | 0 | 0（修复后，见下） |
| Full: max_children_node | 西牛贺洲 | 西牛贺洲 |

## 结论（对 C7 的回答）

**279 量级的崩塌是 Claude 抽取风格特异的，没有在 DeepSeek 上复现。**

- DeepSeek 抽取下 greedy voting 的 max_children 只有 56（vs Claude 279），无环，roots 5 个。崩塌方向一致（投票仍产生 56 扇出的 catch-all"天下"、多根、碎片），但**严重度是 extractor-dependent 的**。
- Full pipeline 在两种抽取器下都收敛到合法单根无环树，max_children 46 vs 63——**结构保证的结论跨 LLM 稳健**，这恰是论文的核心主张（guarantee by construction，不依赖抽取器）。
- 对论文 framing 的影响：motivation 不能写成"greedy voting 必然爆炸到数百"，应写成"greedy aggregation 不提供任何结构保证，其失效严重度随抽取器/文本波动（56–279），而本管线无条件保证合法树"。是否改正文见 STATUS.md 决议。

## 副产品：修环 bug 发现与修复（重要）

首轮 (b) 跑出 cycles=1（`黑水河 ↔ 黑水河水府` 2-环，计数函数因上游节点重复访问报 4）。根因：`EdmondsResolver` Phase 3 单趟修环用最弱边替换为 Edmonds 选择，但当 Edmonds 选择恰是环内另一边（name-containment 注入边 votes 权重为 0 → 被选为最弱边；注入权重 25 → Edmonds 全局解保留它）时替换是 no-op，环存活到最终输出。Claude 五本数据从未触发此路径。

- 修复：`_break_cycles_fixpoint` 不动点迭代（候选顺序：Edmonds 选择∉环且祖先链安全 → uber_root → 删边），并在 Phase 4/5 后再跑一遍终检。主仓 `edmonds_resolver.py` + 回归测试 `test_edmonds_cycle_fixpoint.py`（5 例），全量 543 passed。
- 冻结数据不受影响：A/B 验证旧代码西游 full pipeline 同样输出 66（今测漂移 63→66→67 与本修复无关）。
- **论文 §3.3 / alg:edmonds-hybrid 的 "cycle-repair" 描述与修复后实现一致**（"post-pass structural repair to ensure structural soundness"），无需改动表述；匿名仓代码需同步此修复。

## 复现

```bash
cd backend && uv run python scripts/cross_llm_replication.py
# 需要 DB 中存在 DeepSeek 抽取的西游 novel（实验后已删除，数据不在冻结集内）
```
