# Task 3: Multi-Stage CI/CD Pipeline with GitHub Actions

![CI](https://github.com/hafizmazeem/task3-cicd/actions/workflows/ci.yml/badge.svg)
![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-multi--stage-2496ED?logo=docker&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-CI-2088FF?logo=githubactions&logoColor=white)

**Author:** Hafiz Muhammad Azeem ([@hafizmazeem](https://github.com/hafizmazeem))
**Internship:** Progree, DevOps

## Objective
Automate testing and integration on every remote push: pull the code, run a static linter, run the unit tests, build the container image, and log the pipeline status on the run page.

## Pipeline Flow

```
git push origin main
        |
        v
GitHub Actions (ci.yml)
        |
        +-- Pull code            actions/checkout
        +-- Set up Python 3.12
        +-- Install dependencies
        +-- Lint                 ruff check .
        +-- Unit tests           pytest
        +-- Build image          docker build (multi-stage)
        +-- Status summary       $GITHUB_STEP_SUMMARY (always runs)
```

## Pipeline Stages

| Stage | What it does |
|---|---|
| Trigger | Runs automatically on every push to `main`; `workflow_dispatch` adds a manual Run button |
| Pull code | `actions/checkout@v4` fetches the repository onto the runner |
| Lint | `ruff check .` performs static analysis without running the code |
| Unit tests | `pytest` runs `test_app.py`, which calls `/health` and expects HTTP 200 |
| Build image | Builds the multi-stage Dockerfile, tagged with the commit SHA so every image traces back to one commit |
| Status summary | Writes the commit ID and job status to the run page; `if: always()` makes it run even when a step fails |

Steps run in order and the job stops at the first failure, so a lint or test error blocks the image build.

## Project Structure

```
task3-cicd/
├── .github/workflows/ci.yml
├── app/
│   ├── app.py
│   ├── test_app.py
│   ├── requirements.txt
│   ├── Dockerfile
│   └── .dockerignore
└── .gitignore
```

## The Application and Dockerfile
A Flask app that counts visits in Redis, served by Gunicorn on port 8000, with a `/health` endpoint. The Dockerfile has two stages: the builder installs the Python packages, and the final slim stage copies only the installed packages and the app code, then runs as the non-root user `appuser`.

## Run Locally

```bash
git clone https://github.com/hafizmazeem/task3-cicd.git
cd task3-cicd/app
pip install -r requirements.txt ruff pytest
ruff check .
pytest -q
docker build -t myapp:local .
```

## Result
Every push to `main` automatically lints, tests and builds the application, and the run page shows the commit ID and final status. The pipeline run for this repository finished with status **success**.
