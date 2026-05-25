# ETL Pipeline — Ecommerce Events (Airflow + Postgres)

An end-to-end batch pipeline that simulates ecommerce clickstream data, lands it in a **raw** layer, builds **marts**, and optionally writes purchase reports. Built as a portfolio / learning project with production-style patterns: idempotent ingest, separated business logic, Dockerized runtime, unit tests, and GitHub Actions CI.

## Architecture

<img width="2750" height="920" alt="image" src="https://github.com/user-attachments/assets/56b3410c-53ec-4014-b87b-8c3d39515fd4" />


## Prerequisites

- Docker Desktop (or Docker Engine + Compose v2)
- Python 3.10+ (optional, for local unit tests)
- ~4 GB free disk for images

## Quick start

From the project root:

```bash
# 1. Build images
docker build -t etl-airflow:local .
docker build -t etl-generator:local ./generator

# 2. Initialize Airflow DB + warehouse schemas
docker compose up -d postgres
docker compose run --rm airflow-init

# 3. Start the stack
docker compose up -d

# 4. Open Airflow UI
open http://localhost:8080   # login: admin / admin
```

Both DAGs are unpaused by default (`DAGS_ARE_PAUSED_AT_CREATION=false`). Within 1–2 minutes you should see green runs and rows in `raw.events`.

### Verify data

```bash
docker compose exec postgres psql -U airflow -d airflow -c "SELECT COUNT(*) FROM raw.events;"
docker compose exec generator ls /data/events/raw | head
```

Reports (when purchases occur in the interval) appear under `./reports/` on the host.

### Stop

```bash
docker compose down      # keep data volumes
docker compose down -v   # wipe Postgres + event files
```

## Unit tests (local, no Docker)

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements-dev.txt
pytest tests/ -v --cov=event_validation --cov=event_ingest --cov-fail-under=90
```

Business logic is intentionally separated from Airflow so tests stay fast and do not require a running scheduler.

## CI/CD

On push/PR to `main` or `master`, [`.github/workflows/ci.yml`](.github/workflows/ci.yml) runs:

1. **unit-tests** — pytest with ≥90% coverage on `event_validation` and `event_ingest`
2. **docker-airflow** — build images, `airflow-init`, `airflow dags list-import-errors`

Reproduce locally:

```bash
pip install -r requirements-dev.txt && pytest tests/ -v --cov=event_validation --cov=event_ingest --cov-fail-under=90
# plus the docker commands from the CI job (see workflow file)
```
## 🧱 STACK
- Airflow
- Phyton
- Docker Compose
- GitHub Actions CI
- pytest
- PostgreSQL
