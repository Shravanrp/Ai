# Revenue Projection Service — Capstone (AAVAIL)

## Project overview

A Flask-based service that predicts next-month revenue (global or per-country) using historical transaction-level data. The model is trained on transactional data across countries and limited to the top 10 revenue countries. The repository contains model code, unit tests, logging, Dockerfile, and CI/test scripts.

## Peer-review checklist (how this repo satisfies each item)

* **Are there unit tests for the API?**
  `tests/test_api.py` (run via `pytest tests/test_api.py`)

* **Are there unit tests for the model?**
  `tests/test_model.py` (unit tests for feature transformations and model predictions)

* **Are there unit tests for the logging?**
  `tests/test_logging.py` (ensures logs are produced to `logs/test.log` when run in test mode)

* **Can all unit tests be run with a single script and do all pass?**
  `scripts/run_all_tests.sh` (runs lint + pytest). Example: `./scripts/run_all_tests.sh`

* **Is there a mechanism to monitor performance?**
  `monitoring/` contains:

  * `drift_detector.py` (periodic drift checks)
  * `performance_dashboard/` (notebooks and scripts for visualizing metrics)
  * `README_MONITORING.md` explains how to run the monitors locally or in a scheduled job.

* **Was there an attempt to isolate read/write unit tests from production models and logs?**
  Test-run uses env var `APP_ENV=test` and writes test artifacts to `tmp_test/`. See `tests/conftest.py` for fixtures.

* **Does the API work as expected?**

  * Endpoint: `/predict` accepts JSON: `{ "target_date": "YYYY-MM-DD", "country": "UK" }` (country optional).
  * See `app/routes.py` and `tests/test_api.py` for examples.

* **Does data ingestion exist as a function or script?**
  `ingestion/ingest.py` implements `ingest_transactions(path)` and `scripts/ingest.sh` demonstrates usage.

* **Were multiple models compared?**
  `models/` includes `baseline_model.pkl`, `model_v1.pkl`, and notebooks `experiments/compare_models.ipynb` used in EDA/model selection.

* **Did the EDA investigation use visualizations?**
  `eda/` contains `eda_plots.ipynb` and PNG exports `eda/plots/*.png`.

* **Is everything containerized within a working Docker image?**

  * `Dockerfile` present.
  * Build & run: `docker build -t aavail-revenue . && docker run -p 5000:5000 aavail-revenue`.

* **Did they use a visualization to compare their model to the baseline model?**
  `experiments/compare_models.ipynb` shows RMSE/MAE/time-series comparison plots and exports `experiments/plots/compare_baseline.png`.

## Repo structure (recommended)

```
README.md
Dockerfile
docker-compose.yml (optional for local stack)
app/
  __init__.py
  routes.py
  model.py
  utils.py
ingestion/
  ingest.py
data/
  README.md (NOT containing raw secret data)
models/
  baseline_model.pkl
  model_v1.pkl
tests/
  test_api.py
  test_model.py
  test_logging.py
scripts/
  run_all_tests.sh
  ingest.sh
monitoring/
  drift_detector.py
  README_MONITORING.md
eda/
  eda_plots.ipynb
experiments/
  compare_models.ipynb
  plots/
logs/  (production logs -- **do not store secrets here**)
tmp_test/ (gitignored; used only for test artifacts)
.env.example
requirements.txt
```

## How to run locally (development)

1. Create a virtualenv and install:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

2. Start Flask app:

```bash
export FLASK_APP=app
export APP_ENV=dev
flask run --port=5000
# or
python -m app
```

Request example:

```bash
curl -X POST http://127.0.0.1:5000/predict \
  -H "Content-Type: application/json" \
  -d '{"target_date":"2025-12-01","country":"United Kingdom"}'
```

## Run all tests (single script)

```bash
chmod +x scripts/run_all_tests.sh
./scripts/run_all_tests.sh
```

`run_all_tests.sh` should:

* set `APP_ENV=test`
* run `pytest --maxfail=1 --disable-warnings -q`
* exit non-zero if any test fails

## Docker (build and run)

```bash
docker build -t aavail-revenue .
docker run -e APP_ENV=prod -p 5000:5
```
