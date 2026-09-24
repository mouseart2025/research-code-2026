# Anonymous Submission

Code and data artifacts for an **anonymous submission** currently under double-blind review. Reviewers reach this repository via the URL embedded in the paper PDF.

All identifying information has been removed.

---

## Layout

```
backend/
  src/                       Python source (FastAPI + extraction pipeline)
  scripts/                   Evaluation and baseline scripts
  tests/                     Automated test suite
  data/
    hierarchy_validation/    Per-novel errata gold (JSON)
    review/                  Per-novel review artifacts
    zh_geonames.tsv          Chinese place name index
  pyproject.toml
frontend/
  src/                       React + TypeScript UI
  package.json
paper/
  latex/main.tex             Paper source (this submission)
  evaluation/v071/           Frozen evaluation JSONs (benchmarks, fair, ablation, baselines)
  evaluation/cross-genre-anna-2026-09-24.json   Anna Karenina boundary-case metrics
  version-refresh-trialrun-2026-09-23.json      Refreshed (v0.78, pure-start) run outputs
  data/frozen_extractions.db  Frozen chapter-level extractions (5 classical novels, sqlite)
LICENSE                      AGPL v3
```

**Version note.** This snapshot is at **v0.78**. The paper's headline numbers are produced by the
v0.78 aggregation pipeline over frozen chapter-level extractions (see "Extraction provenance"
below); the frozen v0.71 evaluation JSONs are retained for provenance of the dagger-marked rows.

---

## Quick Start

Environment: Python ≥ 3.9, Node.js ≥ 22, [uv](https://docs.astral.sh/uv/), [Ollama](https://ollama.com/) or an OpenAI-compatible API key.

```bash
cd backend && uv sync && uv run uvicorn src.api.main:app --reload
cd frontend && npm install && npm run dev
```

Backend: http://localhost:8000 · Frontend: http://localhost:5173

---

## Reproducing tables

### Number audit (regenerates every tabular claim from the released JSON)

```bash
cd backend
ARBOR_PAPER_ROOT=$PWD/../paper uv run python scripts/audit_paper_numbers.py
```

Expected output: `Summary: 34 PASS, 0 FAIL`. The auditor parses `paper/latex/main.tex` and
recomputes every numeric claim from `paper/evaluation/v071/*.json` (frozen rows) and
`paper/version-refresh-trialrun-2026-09-23.json` (refreshed rows), including the refreshed
main table (per-novel Overall + error counts), the fair-comparison table
(full 0.9101 vs voting 0.9062 macro averages), and the structural ablation
(Journey 203→57 max_children, pure-start).

### Full-pipeline re-derivation (deeper level)

`paper/data/frozen_extractions.db` is a self-contained sqlite with the frozen chapter-level
extractions (5 classical novels, 552 chapter fact records). To re-run the v0.78 aggregation
pipeline from a pinned empty start state (tier→votes→prior→edmonds→auditor→suffix→purify):

```bash
cd backend
VR_BASE_DB=$PWD/../paper/data/frozen_extractions.db VR_DATA_DIR=/tmp/arbor-pure-start \
  uv run python scripts/version_refresh_trialrun.py --pure-start --run
```

The script clears `world_structures`/`hierarchy_snapshots` for the five novels inside the
scratch copy (the pinned "pure extraction start"), runs the chain twice, and prints a
per-novel determinism report (`ws_edge_diff=0` expected on every line).

### Other baselines

```bash
cd backend
uv run pytest tests/ -x -q                                 # test suite
uv run python scripts/ablation_study.py                    # per-node
uv run python scripts/ablation_hierarchy.py                # structural
uv run python scripts/zero_shot_baseline.py --novel xiyouji
uv run python scripts/single_shot_cot_baseline.py --both
uv run python scripts/graphrag_style_baseline.py           # aggregation baselines
uv run python scripts/eval_contamination_free.py           # synthetic
```

Cloud-LLM baselines require `LLM_API_KEY` in `.env`.

---

## Extraction provenance (read before citing numbers)

The evaluation stack involves three extraction sources, kept distinct on purpose:

1. **Frozen chapter extractions (paper's five classical novels).** The headline evaluation
   uses one frozen extraction set throughout (`paper/data/frozen_extractions.db`,
   `llm_model=MiniMax-M2.7`). The v0.78 refresh changed the *aggregation* pipeline only;
   extractions were not re-generated for the refresh.
2. **Cross-genre novels.** The six contemporary/translated novels in the cross-genre table
   (sci-fi, wuxia, LOTR, realism, xianxia web serial) were extracted with MiniMax-M2.5/2.7
   and are used for structural validation only (no gold).
3. **Anna Karenina boundary case.** Extracted by an in-house direct-extraction agent
   (no external LLM API; `llm_model=kimi-agent`) and merged over the pre-existing
   MiniMax-extracted copy; originals are preserved in `fact_json_original`.

A cross-LLM check with a second vendor's model (DeepSeek V3, July 2026) confirmed the
structural guarantees are extractor-independent (`evaluation/v071/cross-llm/`).

---

## Gold artifacts

Five Chinese classical novels, per-novel errata JSON under `backend/data/hierarchy_validation/`:

- `xiyouji_errata_gold.json`
- `honglou_errata_gold.json`
- `shuihu_errata_gold.json`
- `sanguo_errata_gold.json`
- `fengshen_errata_gold.json`

Each node annotated with entity validity, name accuracy, tier, parent, and structural-error categories.

---

## License

AGPL v3 (see `LICENSE`).
