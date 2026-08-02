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
LICENSE                      AGPL v3
```

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

```bash
cd backend

uv run pytest tests/ -x -q                                 # test suite
uv run python scripts/ablation_study.py                    # per-node
uv run python scripts/ablation_hierarchy.py                # structural
uv run python scripts/pure_mwa_ablation.py                 # Table 8 (pure-MWA ablation)
uv run python scripts/m5_no_edmonds_ablation.py            # post-hoc ablation (voting+repair, no Edmonds)
uv run python scripts/multi_seed_evaluate.py               # extraction stochasticity (multi-seed)
uv run python scripts/bootstrap_ci.py                      # 95% CI on gold Overall (10k stratified bootstrap)
uv run python scripts/zero_shot_baseline.py --novel xiyouji
uv run python scripts/single_shot_cot_baseline.py --both
uv run python scripts/graphrag_style_baseline.py           # aggregation baselines
uv run python scripts/eval_contamination_free.py           # synthetic
uv run python scripts/audit_paper_numbers.py               # number audit
```

Cloud-LLM baselines require `LLM_API_KEY` in `.env`.

---

## Gold artifacts

Five Chinese classical novels, per-novel errata JSON under `backend/data/hierarchy_validation/`:

- `xiyouji_errata_gold.json`
- `honglou_errata_gold.json`
- `shuihu_errata_gold.json`
- `sanguo_errata_gold.json`
- `fengshen_errata_gold.json`

Each node annotated with entity validity, name accuracy, tier, parent, and structural-error categories.

The gold annotations are released under **CC BY 4.0** (see `backend/data/hierarchy_validation/LICENSE`).

---

## License

- Code: **AGPL v3** (see `LICENSE`).
- Gold annotations (`backend/data/hierarchy_validation/`): **CC BY 4.0**.
