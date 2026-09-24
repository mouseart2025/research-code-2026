# Canonical Ablation — v0.71.x Fixes

Qualitative ablation showing per-fix contribution to canonical correctness.
Combines audit findings (cross-novel diagnostics 2026-04-11) with fix commit history.

## 西游记

| Canonical Group | Before v0.71 | After v0.71.2 | Fix attribution |
|---|---|---|---|
| 唐僧 group | **陈玄奘** (freq=87) | **三藏** (freq=1325) | B1: 3-char dominance ratio 10× |
| 八戒 group | **那呆子** (freq=145) | **八戒** (freq=1673) | Phase A B2: override restriction + nickname extension |
| 沙僧 group | split: {沙僧+沙和尚} ∥ {卷帘大将+沙悟净+7} | **沙僧** (merged) | Phase C person_knowledge_prior |
| 孙悟空 group | split into 4 groups ({孙行者…}∥{石猴…}∥{猴精}∥{齐天大圣}) | **孙悟空** (all merged) | Phase C prior + B5 unknown rescue |
| 观音菩萨 group | {观音菩萨}∥{观世音菩萨} (singletons) | **观音菩萨** (merged) | Phase C prior |
| 牛魔王 group | **大力王** (non-standard) | **牛魔王** (canonical) | Phase C prior |
| 玉帝 group | (separate) | **玉帝** ← 天尊/玉皇大帝 | Phase C prior |
| 白龙马 group | (separate) | **白龙马** ← 玉龙/龙马/三太子 | Phase C prior |

## 红楼梦

| Canonical Group | Before | After v0.71.2 | Fix attribution |
|---|---|---|---|
| 贾母 | **史老太君** (split 贾母 ∥ 史老太君 ∥ 老太太) | **贾母** ← 史老太君 | Phase C prior + B6 blocklist (太太/老太太) |
| 贾宝玉 | split: 贾宝玉(98ch) ∥ 宝玉(18ch) | **宝玉** (merged 116ch) | **B3 Layer 0.5 substring exception** |
| 贾探春/惜春/迎春 | split: 贾X ∥ X (each) | **探春/惜春/迎春** (merged) | **B3 substring exception** |
| 薛宝钗 | split: 薛宝钗(4) ∥ 宝钗(94) | **宝钗** (merged 98ch) | B3 substring |
| 王熙凤 | split: 王熙凤 ∥ 凤姐(94) | **凤姐儿** ← 凤姐/王熙凤 | Phase C prior (evidence accumulation) |
| 李纨 | **李氏** (capturing 宫裁/稻香老农) | **李纨** (correct canonical) | B6 "X氏" blocklist + prior |
| Fake characters | **太太** / **老太太** / **贾母等** as primaries | all filtered | Phase A B2 + visualization post-filter |

## Location Merges (via SuffixNormalizer)

| Base | Merged variants | Attribution |
|---|---|---|
| 南赡部洲 | 南瞻部洲, 南膳部洲 | S8 char variant normalization |
| 乌斯藏国 | 乌斯藏, 乌斯藏界, 乌斯藏国界 | S2 SuffixNormalizer explicit synonyms |
| 朱紫国 / 车迟国 / 乌鸡国 / 比丘国 | X城池 variants (8+) | S2 suffix stripping + S6 rule 20 |
| 玉华县 | 玉华城, 玉华州城池, 玉华州城头 | S2 explicit + S6 |
| 平顶山 / 五行山 | X山路, X山顶 | S2 + S6 rule 21 |
| 都中 (红楼京城) | 石头城, 金陵, 神京, 京都, 京师 | **Phase C location prior (new)** |
| 贾母院 (红楼) | 贾母处, 贾母房, 贾母上房, 贾母里间 | S2 explicit synonym |

## Pipeline Stage Contribution (structural, 5-novel avg)

From `ablation-by-stage.json`:

| Stage | avg_depth Δ | max_children Δ | Effect |
|---|---|---|---|
| import (LLM raw) | — | — | baseline |
| **tier** | +0.59 | **-50% max_ch** | biggest single contribution; forces hierarchy coherence |
| votes | 0 | 0 | neutral (only accumulates votes, doesn't change parents) |
| prior | 0 | 0 | neutral structurally (injects votes for Edmonds) |
| edmonds | +0.19 | -4 | resolves orphans + fixes cycles |
| **suffix** | -0.1 (honglou -0.45!) | -1 to -2 | variant merges; depth drops when deep alias-chains collapse |

## Filter / Pre-extraction Contribution (qualitative)

Catch-all parent nodes eliminated from hierarchy (xiyouji, S4 phantom lift):

| Before v0.71.1 | Children | mention | After v0.71.2 |
|---|---|---|---|
| 陷空山 | 29 | 3 | Reduced to 13 via phantom lift ratio guard |
| 盘丝岭 | 19 | 1 | Reduced to 6 |
| 柴扉 (literally "firewood gate") | 9 | 0 | **Eliminated** (filter Rule 20) |
| 海州 | 10 | 3 | **Eliminated** |
| 哈咇国 | 8 | 0 | **Eliminated** |

Generic location names filtered by expanded `_GENERIC_FACILITY_NAMES`:
- 高山, 洞府, 荒野, 假山, 山石, 池中, 西行路上, 东南角井边

Generic character names filtered by `_GENERIC_PERSON_WORDS`:
- 太太, 老太太, 奶奶, 嬷嬷, 群猴, 五百阿罗, 夜叉, 土地神祗, 樵子

## Key Quantitative Summary

| Metric | v0.69 | v0.71.2 | Change |
|---|---|---|---|
| 5-novel gold Overall | 0.98 (3 novels avg) | 0.93 (5 novels avg) | Extended coverage; 水浒/三国 tier errors drag avg |
| Canonical correctness (Canonical hit rate top 10) | ~80% | **~95%** (live graph) | — |
| Catch-all parent nodes | 27 | **19** (-30%) | S4 phantom lift |
| Hierarchy suffix variants | 60+ unmerged | **fully merged** | S2 SuffixNormalizer |
| Tests | 407 | **483** (+19%) | New: test_edmonds_phantom_lift (+6) |
