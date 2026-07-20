# 匿名投稿代码库

当前处于双盲评审期的匿名投稿配套代码。审稿人通过 paper PDF 中的 URL 访问本仓库。

所有可识别信息已清除。完整说明请参见 [README.md](./README.md)（英文版）。

## 目录速览

| 路径 | 说明 |
|------|------|
| `backend/src/` | 后端源码（FastAPI + 抽取管线） |
| `backend/scripts/` | 评测与 baseline 脚本 |
| `backend/tests/` | 自动化测试 |
| `backend/data/hierarchy_validation/` | 五本小说 errata gold JSON |
| `frontend/src/` | 前端（React + TypeScript） |
| `LICENSE` | AGPL v3 |

## 快速开始

```bash
cd backend && uv sync && uv run uvicorn src.api.main:app --reload
cd frontend && npm install && npm run dev
```

## 复现实验

```bash
cd backend
uv run pytest tests/ -x -q
uv run python scripts/ablation_study.py
uv run python scripts/ablation_hierarchy.py
uv run python scripts/zero_shot_baseline.py --novel xiyouji
uv run python scripts/single_shot_cot_baseline.py --both
uv run python scripts/graphrag_style_baseline.py
uv run python scripts/eval_contamination_free.py
uv run python scripts/audit_paper_numbers.py
```
