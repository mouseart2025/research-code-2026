# LLM-CoT Baseline — Error Pattern Analysis

> 2026-04-17 | 从 single_shot_cot/{xiyouji,honglou}.json 提取的具体失败模式。
> 用于 paper/latex/main.tex 的 Table 7 caption 和 §3 叙事支撑。

## 定量对比

| 指标 | 西游 LLM-CoT | 红楼 LLM-CoT | 西游 Full | 红楼 Full | Edmonds 一致性 |
|------|-----|-----|-----|-----|------|
| max_ch | 74 | **143** | 63 | 76 | LLM variance=69, Edmonds variance=13 |
| roots | 2 | **6** (stated 7) | 1 (天下) | 1 | Edmonds guarantees ≤1 |
| missed | 6 | 1 | 0 | 0 | Edmonds 不会丢 |
| halluc | 10 | 3 | 0 | 0 | Edmonds 不会造 |
| parent_P | 0.783 | 0.652 | ~0.998 | ~0.994 | gap 21-34pp |
| chain_acc | 0.642 | **0.323** | — | — | 红楼 2/3 链条错 |

## 失败模式分类(可写入论文)

### 模式 A:跨小说知识污染(LLM 幻觉)
**最震撼的单一证据**:LLM-CoT 在处理《红楼梦》时,把《西游记》的四大部洲作为顶级根节点加了进来:

> 红楼梦 stated_roots = ['东胜神洲', '西牛贺洲', '南赡部洲', '北俱芦洲', '主世界', '天下', '世界']

"东胜神洲/西牛贺洲/南赡部洲/北俱芦洲" **不属于红楼梦** — LLM 的预训练记忆把西游的世界观泼进了红楼的 prompt。这是 **contamination-as-hallucination** 的直接证据,与 §3.1 "LLM pretraining contamination 创造 ceiling effect" 形成有趣的反转:预训练知识有时不是天花板,而是主动污染源。

### 模式 B:后缀变体幻觉(LLM 造词)
西游 LLM-CoT 的幻觉里有:
- 宝象国城池、比丘国城池、灭法国城池(输入只有"宝象国"、"比丘国"、"灭法国")
- 西梁国界(输入只有"西梁女国")

这直接攻击 Edmonds+Prior 附带的 **SuffixNormalizer** 的必要性 — 它主动合并的那些变体,LLM 在没有规则约束时会**反向主动制造**。

### 模式 C:场景引用误作地点(细粒度 entity drift)
红楼 LLM-CoT 把"周瑞家的房中"、"秦氏房中"当成具体命名地点;西游 LLM-CoT 造出"老者家中"、"山凹"、"龙床"、"中堂"。

这呼应了 §2.3 Layer 2(领域知识规则)中"人名+处/房 → 地点"的 context-aware 规则 — LLM 没有这个规则,产生**过度提取**。

### 模式 D:结构坍缩(LLM 无约束时的 fan-out 爆炸)
- 红楼 大观园 = 143 个子节点(Full pipeline = 76)
- 西游 西牛贺洲 = 74 个子节点(Full pipeline 也是 63 — 接近)

有趣的不对称:西游的最高 fan-out 节点 LLM 和 Edmonds 差距小(74 vs 63),而红楼的差距大(143 vs 76)。说明 LLM 在"地理清晰、等级明确"的小说(西游)上接近 Edmonds,但在"结构复杂、同级多样"的小说(红楼)上严重退化。这是**结构保证价值的异质性**,可作为论文讨论细节。

### 模式 E:多根问题(LLM 无 single-root 约束)
红楼 LLM-CoT 报 7 个 stated roots + 实际 6 个 computed roots(石头城/神京/都中/太虚幻境/城外原乡 等并列),Edmonds 保证合并到单一 uber-root。

## 论文 Table 7 caption 建议(覆盖上面所有模式)

> Table 7. Structural properties of five hierarchy-construction methods on *Journey to the West* and *Dream of the Red Chamber*. LLM-CoT receives the same vote evidence Edmonds sees (location set + per-location parent-candidate counts) and is asked in a single Claude Sonnet 4 call to emit the full containment tree as JSON. Despite access to pretrained knowledge and global visibility, LLM-CoT exhibits qualitative failures that algorithmic aggregation prevents by construction: (a) **data corruption** — 6/1 input locations dropped and 10/3 hallucinated per novel, including cross-novel pollution (LLM placed *Journey*'s four continents as roots of *Red Chamber*'s hierarchy); (b) **structural inconsistency** — max fan-out varies 74→143 across two novels (variance 69), versus Edmonds's 63→76 (variance 13); (c) **no structural guarantees** — *Red Chamber* produced 6 disjoint roots where a valid tree has 1. Even when LLM-CoT's output happens to be acyclic (as on *Journey*), this is coincidence, not proof.

## 关键数字冲突(继续待核实)

`ablation-by-stage.json` 的 **西游 Edmonds+Prior max_ch = 63**,而 `main.tex` 当前写的是 **52**。差 11。需要在投稿前回溯并统一。
