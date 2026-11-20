<h1 align="left">Day 05 – Python for DevOps + Mini Project 1 Start – Commands & Setup Steps</h1>

<p>
<img src="https://img.shields.io/badge/Week-02-1F3864?style=flat-square" alt="Week: 02">
<img src="https://img.shields.io/badge/Day-05-2E75B6?style=flat-square" alt="Day: 05">
<img src="https://img.shields.io/badge/Cloud-Local-FF9900?style=flat-square" alt="Cloud: Local">
<img src="https://img.shields.io/badge/Class-8%3A00%E2%80%9312%3A15%20EST-2E75B6?style=flat-square" alt="Class: 8:00–12:15 EST">
<img src="https://img.shields.io/badge/Lab-1%3A15%E2%80%935%3A00%20EST-2E7D32?style=flat-square" alt="Lab: 1:15–5:00 EST">
</p>

<p><a href="Day05_Python_FastAPI_and_MiniProject1_Start.md">Notes</a> · <b>Commands & Setup</b> · <a href="Day05_Questions_and_Answers.md">Questions & Answers</a> · <a href="../README.md">Week overview</a> · <a href="../../README.md">Course index</a></p>



Companion to the main notes: [Day05_Python_FastAPI_and_MiniProject1_Start.md](Day05_Python_FastAPI_and_MiniProject1_Start.md). Every command used today, in the order you meet it in class and lab,
with the command breakdown tables that explain each part. Run commands in your own terminal and replace
`<your-name>` with your resource name from the Batch Roster.


<h2>1. Installation & Setup Steps</h2>


Run these before or at the start of the lab (installs, logins, cluster start-up, tool configuration).

<b>From:</b> 1:15 – 2:45 PM · Lab Part 1 – Task Assignment & Guided Build → Task 1 – Branch, Seed Data, venv and Requirements

```bash
cd ~/devops-8wk/devops-8wk-<your-name>
git switch main && git pull
git switch -c day05-python-fastapi
mkdir -p app tests data
# same five employees as the team repo (Day 4)
cp ~/devops-8wk/peoplepulse-team/data/employees.csv data/employees.csv 2>/dev/null || cat > data/employees.csv <<'EOF'
id,name,email,department,title,location,hire_date
1,Haiman Sher,haiman.sher@example.com,Platform,DevOps Engineer,Austin,2023-01-09
2,Cindia Fouda,cindia.fouda@example.com,Data,Data Analyst,Chicago,2022-07-18
3,Jobanjit Singh,jobanjit.singh@example.com,Security,Cloud Security Engineer,Seattle,2021-11-01
4,Carla Rivera,carla.rivera@example.com,QA,QA Engineer,Dallas,2024-03-11
5,Raymundo Trancoso,raymundo.trancoso@example.com,Platform,SRE,Denver,2024-05-20
EOF

cat >> .gitignore <<'EOF'
# Python
.venv/
__pycache__/
*.pyc
.pytest_cache/
.ruff_cache/
.coverage
htmlcov/
reports/
EOF

python3 -m venv .venv && source .venv/bin/activate
python -m pip install --upgrade pip
printf 'fastapi==0.115.6\nuvicorn[standard]==0.34.0\n' > requirements.txt
printf -- '-r requirements.txt\npytest==8.3.4\npytest-cov==6.0.0\nhttpx==0.28.1\nruff==0.8.4\n' > requirements-dev.txt
python -m pip install -r requirements-dev.txt
git status --short
```


<h2>2. Commands by Session</h2>


<h3>8:15 – 9:45 AM · Concept Session 1</h3>


<h4>1. Why Python for DevOps</h4>


```bash
# Bash: fine for one URL
curl -fsS http://127.0.0.1:8000/health > /dev/null && echo "healthy"
```

<h4>3. Virtual Environments (venv)</h4>


```bash
cd ~/devops-8wk/devops-8wk-<your-name>
python3 -m venv .venv                  # create the venv once per project
source .venv/bin/activate              # activate it (Windows PowerShell: .venv\Scripts\Activate.ps1)
which python                           # now points into .venv
python -m pip install --upgrade pip    # upgrade pip inside the venv
python -m pip --version
deactivate                             # leave the venv when you are done
```

Expected output:

```text
/home/haiman/devops-8wk/devops-8wk-haiman-sher/.venv/bin/python
pip 24.3.1 from /home/haiman/devops-8wk/devops-8wk-haiman-sher/.venv/lib/python3.12/site-packages/pip (python 3.12)
```

<b>Command breakdown:</b> `python3 -m venv .venv && source .venv/bin/activate`

| Part | What it means |
|---|---|
| `python3` | the system Python interpreter |
| `-m venv` | run the built-in module `venv` as a program |
| `.venv` | the folder to create; the leading dot hides it, and our `.gitignore` already ignores it |
| `source` | run the activate script <b>in the current shell</b>, so it can change your `PATH` |
| `.venv/bin/activate` | puts `.venv/bin` first in `PATH`, so `python` and `pip` now mean the venv copies; your prompt shows `(.venv)` |
| `deactivate` | restore the old `PATH` |

<h4>4. pip, Requirements Files and Version Pinning</h4>


```bash
python -m pip install -r requirements-dev.txt   # install everything from the list
python -m pip show fastapi | head -3             # what is installed, and which version
python -m pip list --outdated                    # what could be upgraded (review, then pin)
python -m pip freeze > /tmp/freeze.txt           # snapshot of ALL installed packages, incl. sub-dependencies
```

Expected output:

```text
Name: fastapi
Version: 0.115.6
Summary: FastAPI framework, high performance, easy to learn, fast to code, ready for production
```

<b>Command breakdown:</b> `python -m pip install -r requirements-dev.txt`

| Part | What it means |
|---|---|
| `python -m pip` | run pip that belongs to the active Python (the venv) |
| `install` | download from PyPI and install into the venv |
| `-r requirements-dev.txt` | read the package list from this file (`-r` = requirements); the file itself includes `requirements.txt` |
| `uvicorn[standard]` (inside the file) | the package plus its optional "standard" extras (faster event loop, auto-reload support) |
| `pip freeze` | print every installed package with its exact version – useful to compare two machines |

<h4>5. Reading and Writing JSON</h4>


<b>Command breakdown:</b> `json.dumps(report, indent=2, sort_keys=True)`

| Part | What it means |
|---|---|
| `json.dumps` | "dump to string": convert a Python object into JSON text |
| `report` | the dict to convert; only plain types (dict, list, str, int, float, bool, None) are allowed |
| `indent=2` | pretty-print with 2 spaces, readable for humans |
| `sort_keys=True` | keys in alphabetical order, so the file is identical every run (clean diffs) |
| `encoding="utf-8"` (on write) | always set it; the default differs between Windows and Linux |

<h4>7. Reading and Writing CSV – the PeoplePulse Data Source</h4>


<b>Command breakdown:</b> `csv.DictWriter(f, fieldnames=["id", "name", "email"], extrasaction="ignore")`

| Part | What it means |
|---|---|
| `open(..., newline="")` | let the `csv` module handle line endings; without it you get blank lines on Windows |
| `csv.DictReader(f)` | read rows as dicts; keys come from the header line |
| `reader.fieldnames` | the list of header columns – use it to check required columns |
| `csv.DictWriter(f, fieldnames=[...])` | write dicts as rows, with these columns in this order |
| `extrasaction="ignore"` | silently drop keys not in `fieldnames` (default `"raise"` fails on extra keys) |
| `writeheader()` / `writerows()` | write the header line, then every row |

<h4>8. Script Patterns for DevOps</h4>


```bash
python3 inventory.py --env prod; echo "exit=$?"
AWS_REGION=eu-west-1 python3 inventory.py --env prod; echo "exit=$?"
python3 inventory.py --env staging; echo "exit=$?"
```

Expected output:

```text
2026-11-13 08:40:02,114 INFO inventory env=prod region=us-east-1
exit=0
2026-11-13 08:40:03,502 INFO inventory env=prod region=eu-west-1
2026-11-13 08:40:03,502 ERROR inventory prod must run in us-east-1, got eu-west-1
exit=1
usage: inventory.py [-h] [--env {dev,prod}] [--verbose]
inventory.py: error: argument --env: invalid choice: 'staging' (choose from 'dev', 'prod')
exit=2
```

<h3>10:00 – 11:15 AM · Concept Session 2 + Live Demo</h3>


<h4>9. HTTP APIs, FastAPI and uvicorn</h4>


```bash
uvicorn demo:app --reload --host 127.0.0.1 --port 8000
# in a second terminal:
curl -s 'http://127.0.0.1:8000/servers?env=prod'; echo
curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8000/servers/99
curl -s http://127.0.0.1:8000/servers/abc; echo
```

Expected output:

```text
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
[{"id":2,"name":"web-2","env":"prod"}]
404
{"detail":[{"type":"int_parsing","loc":["path","server_id"],"msg":"Input should be a valid integer, unable to parse string as an integer","input":"abc"}]}
```

<b>Command breakdown:</b> `uvicorn demo:app --reload --host 127.0.0.1 --port 8000`

| Part | What it means |
|---|---|
| `uvicorn` | the ASGI web server (ASGI = the standard way Python async web apps talk to servers) |
| `demo:app` | "in the file `demo.py`, use the object named `app`"; for our project it is `app.main:app` |
| `--reload` | restart automatically when you save a file – for development only, never in production |
| `--host 127.0.0.1` | listen only on loopback: reachable from this machine only |
| `--port 8000` | TCP port; PeoplePulse always uses 8000 in this course |

<b>Command breakdown:</b> `curl -s -o /dev/null -w '%{http_code}\n' URL`

| Part | What it means |
|---|---|
| `-s` | silent: no progress bar |
| `-o /dev/null` | throw the response body away |
| `-w '%{http_code}\n'` | "write out" only the HTTP status code, then a newline |

<h4>11. The PeoplePulse Project Layout and Configuration</h4>


```bash
python -c "from app.config import get_data_file; print(get_data_file())"
DATA_FILE=/tmp/other.csv python -c "from app.config import get_data_file; print(get_data_file())"
```

Expected output:

```text
/home/cindia/devops-8wk/devops-8wk-cindia-fouda/data/employees.csv
/tmp/other.csv
```

<b>Command breakdown:</b> `Path(__file__).resolve().parent.parent / "data" / "employees.csv"`

| Part | What it means |
|---|---|
| `__file__` | the path of the current file, `app/config.py` |
| `.resolve()` | turn it into a full absolute path |
| `.parent.parent` | go up two levels: from the file to `app/`, then to the repo root |
| `/ "data" / "employees.csv"` | `pathlib` joins paths with `/` – works on Linux, macOS and Windows |
| `os.getenv("DATA_FILE", default)` | use the env var if it is set, otherwise the default |

<h4>12. pytest – Unit Tests, API Tests and Fixtures</h4>


```bash
pytest -v
```

Expected output:

```text
tests/test_demo.py::test_list_servers_filters_by_env PASSED              [ 25%]
tests/test_demo.py::test_get_server_status[1-200] PASSED                 [ 50%]
tests/test_demo.py::test_get_server_status[2-200] PASSED                 [ 75%]
tests/test_demo.py::test_get_server_status[99-404] PASSED                [100%]
4 passed in 0.21s
```

<b>Command breakdown:</b> useful pytest options

| Part | What it means |
|---|---|
| `pytest` | find and run all tests (settings come from `pyproject.toml`) |
| `-v` | verbose: one line per test, including each parametrize case |
| `-q` | quiet: dots only – good for CI logs |
| `-k records` | run only tests whose name contains "records" |
| `-x` | stop at the first failure |
| `--lf` | re-run only the tests that failed last time |

<h4>13. Code Coverage and the 80% Gate</h4>


```bash
pytest --cov=app --cov-report=term-missing --cov-fail-under=80
```

Expected output:

```text
Name                Stmts   Miss Branch BrPart  Cover   Missing
---------------------------------------------------------------
app/__init__.py         2      0      0      0   100%
app/config.py           5      0      0      0   100%
app/main.py            21      2      2      0    91%   19-20
app/models.py          10      0      0      0   100%
app/repository.py      20      3      6      1    85%   26, 32-33
---------------------------------------------------------------
TOTAL                  58      5      8      1    91%

Required test coverage of 80% reached. Total coverage: 90.91%
5 passed in 0.42s
```

<b>Command breakdown:</b> `pytest --cov=app --cov-report=term-missing --cov-fail-under=80`

| Part | What it means |
|---|---|
| `--cov=app` | measure coverage for the `app` package only (not the tests themselves) |
| `--cov-report=term-missing` | print the table with the `Missing` line numbers in the terminal |
| `--cov-report=html` (alternative) | write a clickable report to `htmlcov/index.html` |
| `--cov-report=xml:reports/coverage.xml` | machine-readable report for Jenkins and GitHub Actions (Day 8) |
| `--cov-fail-under=80` | exit with a failure code when total coverage is below 80% |
| `Branch` / `BrPart` | number of `if` branches / branches only partly tested (needs `branch = true` in config) |

<h4>14. Linting and Formatting with ruff</h4>


```bash
ruff check .            # find problems
ruff check . --fix      # auto-fix the safe ones (import order, unused imports)
ruff format .           # format the code
ruff format --check .   # CI mode: fail if any file would be reformatted
```

Expected output:

```text
app/main.py:3:8: F401 [*] `os` imported but unused
Found 1 error.
[*] 1 fixable with the `--fix` option.
```

<h4>15. boto3 – Python Automation for AWS</h4>


<b>Command breakdown:</b> `ec2.get_paginator("describe_instances").paginate(Filters=[...])`

| Part | What it means |
|---|---|
| `boto3.Session(region_name=region)` | a session with a fixed region; credentials come from the default chain |
| `.client("ec2")` | low-level client for the EC2 API (same operations as `aws ec2 ...`) |
| `get_paginator("describe_instances")` | AWS returns results in pages; the paginator fetches every page for you |
| `Filters=[{"Name": "instance-state-name", "Values": ["running"]}]` | server-side filter, same as `--filters` in the CLI |
| `inst.get("Tags", [])` | instances without tags have <b>no</b> `Tags` key – `.get` avoids a `KeyError` |
| `REQUIRED - tags.keys()` | set difference: the required tags that are not present |

<h4>16. The Azure SDK for Python</h4>


```bash
export AZURE_SUBSCRIPTION_ID=$(az account show --query id -o tsv)
python list_rgs_azure.py
```

Expected output:

```text
NetworkWatcherRG               eastus     owner=-
```

<b>Command breakdown:</b> `export AZURE_SUBSCRIPTION_ID=$(az account show --query id -o tsv)`

| Part | What it means |
|---|---|
| `az account show` | show the subscription your `az login` session is using |
| `--query id` | JMESPath filter: only the subscription ID |
| `-o tsv` | plain text without quotes, safe to store in a variable |
| `$( ... )` | command substitution: run it and insert the output |
| `export` | make the variable visible to the Python process you start next |

<h4>Live Demo: From CSV to a Tested API</h4>


```bash
mkdir -p ~/day05-demo && cd ~/day05-demo
python3 -m venv .venv && source .venv/bin/activate
python -m pip install -q fastapi==0.115.6 "uvicorn[standard]==0.34.0" pytest==8.3.4 httpx==0.28.1 PyYAML==6.0.2
python -c "import fastapi, sys; print(fastapi.__version__, sys.version.split()[0])"
```

Expected output:

```text
0.115.6 3.12.3
```

```bash
mkdir -p data && cat > data/employees.csv <<'EOF'
id,name,email,department,title,location,hire_date
1,Haiman Sher,haiman.sher@example.com,Platform,DevOps Engineer,Austin,2023-01-09
2,Cindia Fouda,cindia.fouda@example.com,Data,Data Analyst,Chicago,2022-07-18
EOF
cat > settings.yaml <<'EOF'
service: peoplepulse
port: 8000
EOF
python - <<'EOF'
import csv, json, yaml
cfg = yaml.safe_load(open("settings.yaml", encoding="utf-8"))
rows = list(csv.DictReader(open("data/employees.csv", newline="", encoding="utf-8")))
print(json.dumps({"service": cfg["service"], "port": cfg["port"], "first": rows[0]["name"], "count": len(rows)}))
EOF
```

Expected output:

```text
{"service": "peoplepulse", "port": 8000, "first": "Haiman Sher", "count": 2}
```

<b>Command breakdown:</b> `python - <<'EOF' ... EOF`

| Part | What it means |
|---|---|
| `python -` | read the program from standard input instead of a file |
| `<<'EOF'` | here-document: everything until the line `EOF` is fed to Python |
| quotes around `'EOF'` | the shell does <b>not</b> expand `$` or backticks inside – the code arrives unchanged |

```bash
cat > main.py <<'EOF'
import csv

from fastapi import FastAPI

app = FastAPI(title="PeoplePulse demo")

@app.get("/health")
def health() -> dict:
    return {"status": "ok"}

@app.get("/records")
def records() -> list[dict]:
    with open("data/employees.csv", newline="", encoding="utf-8") as f:
        return list(csv.DictReader(f))
EOF
uvicorn main:app --host 127.0.0.1 --port 8000 > uvicorn.log 2>&1 &
sleep 2
curl -s http://127.0.0.1:8000/health; echo
curl -s http://127.0.0.1:8000/records | python -m json.tool | head -10
```

Expected output:

```text
{"status":"ok"}
[
    {
        "id": "1",
        "name": "Haiman Sher",
        "email": "haiman.sher@example.com",
        "department": "Platform",
        "title": "DevOps Engineer",
        "location": "Austin",
        "hire_date": "2023-01-09"
    },
```

<b>Command breakdown:</b> `uvicorn main:app --host 127.0.0.1 --port 8000 > uvicorn.log 2>&1 &`

| Part | What it means |
|---|---|
| `uvicorn main:app` | serve the `app` object from `main.py` |
| `> uvicorn.log` | send normal output to a log file instead of the terminal |
| `2>&1` | send errors (stderr) to the same place as stdout |
| `&` | run in the background, so the terminal is free for `curl`; stop it later with `kill %1` |
| `python -m json.tool` | pretty-print JSON (works where `jq` is not installed) |

```bash
cat > test_main.py <<'EOF'
from fastapi.testclient import TestClient

from main import app

client = TestClient(app)

def test_health():
    resp = client.get("/health")
    assert resp.status_code == 200
    assert resp.json() == {"status": "ok"}

def test_records_returns_all_rows():
    assert len(client.get("/records").json()) == 2
EOF
pytest -q
```

Expected output:

```text
..                                                                       [100%]
2 passed in 0.31s
```

```bash
sed -i 's/== 2/== 3/' test_main.py && pytest -q
```

Expected output:

```text
F.
    def test_records_returns_all_rows():
>       assert len(client.get("/records").json()) == 3
E       AssertionError: assert 2 == 3
E        +  where 2 = len([{'department': 'Platform', ...}, {...}])
FAILED test_main.py::test_records_returns_all_rows - AssertionError: assert 2 == 3
1 failed, 1 passed in 0.35s
```

```bash
python -m pip install -q boto3==1.35.90
python - <<'EOF'
import boto3
print("Account:", boto3.client("sts").get_caller_identity()["Account"])
ec2 = boto3.client("ec2", region_name="us-east-1")
resp = ec2.describe_instances(Filters=[{"Name": "instance-state-name", "Values": ["running"]}])
print("Running instances:", sum(len(r["Instances"]) for r in resp["Reservations"]))
EOF
```

Expected output:

```text
Account: <ACCOUNT_ID>
Running instances: 0
```

```bash
python -m pip install -q azure-identity==1.19.0 azure-mgmt-resource==23.2.0
export AZURE_SUBSCRIPTION_ID=$(az account show --query id -o tsv)
python - <<'EOF'
import os
from azure.identity import DefaultAzureCredential
from azure.mgmt.resource import ResourceManagementClient
client = ResourceManagementClient(DefaultAzureCredential(), os.environ["AZURE_SUBSCRIPTION_ID"])
print("Resource groups:", len(list(client.resource_groups.list())))
EOF
kill %1    # stop the background uvicorn
```

Expected output:

```text
Resource groups: 1
```

<h3>1:15 – 2:45 PM · Lab Part 1 – Task Assignment & Guided Build</h3>


<h4>Task 1 – Branch, Seed Data, venv and Requirements</h4>


<b>Command breakdown:</b> `cp ... 2>/dev/null || cat > data/employees.csv <<'EOF'`

| Part | What it means |
|---|---|
| `cp <team repo>/data/employees.csv data/` | reuse the seed file from Day 4 if you still have the team repo cloned |
| `2>/dev/null` | hide the "No such file" message if you do not |
| `\|\|` | "or": run the right side only if `cp` failed – then we write the file by hand |
| `printf -- '-r requirements.txt\n...'` | `--` tells `printf` that the text starting with `-r` is not an option |

<h4>Task 2 – Project Configuration (pyproject.toml)</h4>


```bash
cat > pyproject.toml <<'EOF'
[project]
name = "peoplepulse"
version = "0.1.0"
description = "PeoplePulse CSV Records API"
requires-python = ">=3.12"

[tool.pytest.ini_options]
testpaths = ["tests"]
pythonpath = ["."]

[tool.coverage.run]
source = ["app"]
branch = true

[tool.coverage.report]
show_missing = true
fail_under = 80

[tool.ruff]
line-length = 100
target-version = "py312"

[tool.ruff.lint]
select = ["E", "F", "I", "B", "UP"]
EOF
python -c "import tomllib; tomllib.load(open('pyproject.toml', 'rb')); print('valid toml')"
```

<h4>Task 3 – Config, Model and CSV Repository</h4>


```bash
cat > app/__init__.py <<'EOF'
"""PeoplePulse CSV Records API."""

import os

# Docker (Day 6) and CI set APP_VERSION; locally we fall back to the code version.
__version__ = os.getenv("APP_VERSION", "0.1.0")
EOF

cat > app/config.py <<'EOF'
"""Runtime configuration, read from environment variables (12-factor)."""

import os
from pathlib import Path

DEFAULT_DATA_FILE = Path(__file__).resolve().parent.parent / "data" / "employees.csv"

def get_data_file() -> Path:
    """Path of the employees CSV; override it with the DATA_FILE env var."""
    return Path(os.getenv("DATA_FILE", str(DEFAULT_DATA_FILE)))
EOF

cat > app/models.py <<'EOF'
"""API schemas (Pydantic models)."""

from datetime import date

from pydantic import BaseModel

class Employee(BaseModel):
    """One row of data/employees.csv, with real types."""

    id: int
    name: str
    email: str
    department: str
    title: str
    location: str
    hire_date: date
EOF

cat > app/repository.py <<'EOF'
"""Load employee records from a CSV file."""

import csv
from pathlib import Path

REQUIRED_COLUMNS = {"id", "name", "email", "department", "title", "location", "hire_date"}

class DataFileError(Exception):
    """Raised when the data file is missing or invalid."""

def load_records(path: Path) -> list[dict]:
    """Return all rows as dicts with an integer ``id``.

    Raises DataFileError if the file is missing, a column is missing,
    or an id is not a whole number.
    """
    if not path.is_file():
        raise DataFileError(f"Data file not found: {path}")

    with path.open(newline="", encoding="utf-8") as f:
        reader = csv.DictReader(f)
        missing = REQUIRED_COLUMNS - set(reader.fieldnames or [])
        if missing:
            raise DataFileError(f"Missing columns: {', '.join(sorted(missing))}")

        records = []
        for line_no, row in enumerate(reader, start=2):  # line 1 is the header
            try:
                row["id"] = int(row["id"])
            except (TypeError, ValueError) as exc:
                raise DataFileError(f"Invalid id on line {line_no}: {row['id']!r}") from exc
            records.append(row)
    return records
EOF
python -c "from app.repository import load_records; from app.config import get_data_file; r = load_records(get_data_file()); print(len(r), type(r[0]['id']).__name__)"
```

Expected output:

```text
5 int
```

<h4>Task 4 – FastAPI App with /health and /records</h4>


```bash
cat > app/main.py <<'EOF'
"""PeoplePulse FastAPI application."""

from typing import Annotated

from fastapi import FastAPI, HTTPException, Query

from app import __version__
from app.config import get_data_file
from app.models import Employee
from app.repository import DataFileError, load_records

app = FastAPI(title="PeoplePulse", version=__version__)

def _records() -> list[dict]:
    """Load records on every request, so a new DATA_FILE is picked up (fine for a small CSV)."""
    try:
        return load_records(get_data_file())
    except DataFileError as exc:
        raise HTTPException(status_code=503, detail=str(exc)) from exc

@app.get("/health")
def health() -> dict:
    """Liveness check: cheap, no file access."""
    return {"status": "ok", "version": __version__}

@app.get("/records", response_model=list[Employee])
def list_records(
    department: Annotated[str | None, Query(description="Filter by department")] = None,
    limit: Annotated[int, Query(ge=1, le=1000)] = 100,
) -> list[dict]:
    """All employees, optionally filtered by department (case-insensitive)."""
    records = _records()
    if department:
        records = [r for r in records if r["department"].lower() == department.lower()]
    return records[:limit]

# TODO (assessment B1): GET /records/{record_id} -> 200 Employee, 404 {"detail": "Record not found"}
EOF
uvicorn app.main:app --reload --host 127.0.0.1 --port 8000 > uvicorn.log 2>&1 &
sleep 2
curl -s http://127.0.0.1:8000/health; echo
curl -s 'http://127.0.0.1:8000/records?department=platform&limit=2' | python -m json.tool
curl -s -o /dev/null -w '%{http_code}\n' 'http://127.0.0.1:8000/records?limit=0'
```

Expected output:

```text
{"status":"ok","version":"0.1.0"}
[
    {
        "id": 1,
        "name": "Haiman Sher",
        "email": "haiman.sher@example.com",
        "department": "Platform",
        "title": "DevOps Engineer",
        "location": "Austin",
        "hire_date": "2023-01-09"
    },
    {
        "id": 5,
        "name": "Raymundo Trancoso",
        "email": "raymundo.trancoso@example.com",
        "department": "Platform",
        "title": "SRE",
        "location": "Denver",
        "hire_date": "2024-05-20"
    }
]
422
```

<b>Command breakdown:</b> `limit: Annotated[int, Query(ge=1, le=1000)] = 100`

| Part | What it means |
|---|---|
| `limit: int` | a query parameter (`?limit=2`), converted to an integer |
| `Annotated[..., Query(...)]` | attach extra rules to the type without changing the default value |
| `ge=1, le=1000` | "greater or equal 1, less or equal 1000" – `limit=0` gives 422 automatically |
| `= 100` | default when the caller does not send `limit` |
| `response_model=list[Employee]` | FastAPI validates and converts every record through the model before sending it |

<h4>Task 5 – First Tests with Coverage</h4>


```bash
cat > tests/conftest.py <<'EOF'
"""Shared pytest fixtures."""

import pytest
from fastapi.testclient import TestClient

from app.main import app

SAMPLE_CSV = """id,name,email,department,title,location,hire_date
1,Haiman Sher,haiman.sher@example.com,Platform,DevOps Engineer,Austin,2023-01-09
2,Cindia Fouda,cindia.fouda@example.com,Data,Data Analyst,Chicago,2022-07-18
3,Jobanjit Singh,jobanjit.singh@example.com,Security,Cloud Security Engineer,Seattle,2021-11-01
"""

@pytest.fixture
def data_file(tmp_path, monkeypatch):
    """Write a known CSV to a temp folder and point the app at it."""
    path = tmp_path / "employees.csv"
    path.write_text(SAMPLE_CSV, encoding="utf-8")
    monkeypatch.setenv("DATA_FILE", str(path))
    return path

@pytest.fixture
def client(data_file):
    """A test client for the app, already using the sample CSV."""
    return TestClient(app)
EOF

cat > tests/test_repository.py <<'EOF'
import pytest

from app.repository import DataFileError, load_records

def test_load_records_converts_id_to_int(data_file):
    records = load_records(data_file)
    assert len(records) == 3
    assert records[0]["id"] == 1

def test_missing_file_raises(tmp_path):
    with pytest.raises(DataFileError, match="not found"):
        load_records(tmp_path / "nope.csv")
EOF

cat > tests/test_api.py <<'EOF'
def test_health(client):
    resp = client.get("/health")
    assert resp.status_code == 200
    assert resp.json()["status"] == "ok"

def test_list_records_returns_all(client):
    resp = client.get("/records")
    assert resp.status_code == 200
    assert [r["id"] for r in resp.json()] == [1, 2, 3]

def test_list_records_filters_department_case_insensitive(client):
    resp = client.get("/records", params={"department": "data"})
    assert [r["name"] for r in resp.json()] == ["Cindia Fouda"]
EOF
pytest --cov=app --cov-report=term-missing
```

Expected output:

```text
.....                                                                    [100%]
Name                Stmts   Miss Branch BrPart  Cover   Missing
---------------------------------------------------------------
app/__init__.py         2      0      0      0   100%
app/config.py           5      0      0      0   100%
app/main.py            21      2      2      0    91%   19-20
app/models.py          10      0      0      0   100%
app/repository.py      20      3      6      1    85%   26, 32-33
---------------------------------------------------------------
TOTAL                  58      5      8      1    91%

Required test coverage of 80.0% reached. Total coverage: 90.91%
5 passed in 0.42s
```

<b>Command breakdown:</b> the `data_file(tmp_path, monkeypatch)` fixture

| Part | What it means |
|---|---|
| `@pytest.fixture` | turns the function into a fixture; any test that names `data_file` as an argument gets its return value |
| `tmp_path` | built-in fixture: a new empty folder for this test only |
| `monkeypatch.setenv("DATA_FILE", ...)` | set the env var for this test; pytest removes it afterwards |
| `client(data_file)` | a fixture can use another fixture – asking for `client` also creates the sample CSV |
| `conftest.py` | pytest loads it automatically; fixtures in it are shared by all test files |

<h4>Task 6 – Lint, Format, Commit and Push</h4>


```bash
ruff check . --fix
ruff format .
ruff check . && ruff format --check .
kill %1 2>/dev/null; rm -f uvicorn.log
git add .gitignore app tests data pyproject.toml requirements.txt requirements-dev.txt
git commit -m "feat(api): add health and records endpoints with csv repository"
git push -u origin day05-python-fastapi
```

Expected output:

```text
All checks passed!
8 files already formatted
[day05-python-fastapi 4c1d2e7] feat(api): add health and records endpoints with csv repository
 11 files changed, 168 insertions(+)
```

<h3>4:00 – 4:45 PM · Lab Review with Trainer & Teardown</h3>


<h4>Reference Solution (shown after scoring)</h4>


```bash
pytest --cov=app --cov-fail-under=80 | tail -4
```

Expected output:

```text
TOTAL                  64      0     12      0   100%

Required test coverage of 80% reached. Total coverage: 100.00%
15 passed in 0.48s
```

```bash
sudo id peoplepulse 2>/dev/null || sudo useradd --system --no-create-home --shell /usr/sbin/nologin peoplepulse
sudo mkdir -p /opt/peoplepulse
sudo cp -r app data requirements.txt /opt/peoplepulse/
sudo python3 -m venv /opt/peoplepulse/.venv
sudo /opt/peoplepulse/.venv/bin/pip install -q -r /opt/peoplepulse/requirements.txt
sudo chown -R root:peoplepulse /opt/peoplepulse && sudo chmod -R o-rwx /opt/peoplepulse
sudo chmod -R g-w /opt/peoplepulse

sudo tee /etc/systemd/system/peoplepulse.service > /dev/null <<'EOF'
[Unit]
Description=PeoplePulse CSV Records API
After=network-online.target
Wants=network-online.target

[Service]
User=peoplepulse
Group=peoplepulse
WorkingDirectory=/opt/peoplepulse
Environment=DATA_FILE=/opt/peoplepulse/data/employees.csv
ExecStart=/opt/peoplepulse/.venv/bin/uvicorn app.main:app --host 127.0.0.1 --port 8000
Restart=on-failure
RestartSec=3

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable --now peoplepulse
systemctl is-active peoplepulse && curl -s 127.0.0.1:8000/health; echo
```

Expected output:

```text
active
{"status":"ok","version":"0.1.0"}
```

<b>Command breakdown:</b> the key lines of `peoplepulse.service`

| Part | What it means |
|---|---|
| `User=peoplepulse` | run as the service account, never root |
| `WorkingDirectory=/opt/peoplepulse` | uvicorn imports `app.main` from here – wrong folder means `ModuleNotFoundError` |
| `Environment=DATA_FILE=...` | the same env var the code reads; no code change between laptop and server |
| `ExecStart=/opt/peoplepulse/.venv/bin/uvicorn ...` | full path to the venv's uvicorn – systemd does not use your shell's `PATH` or activated venv |
| `Restart=on-failure` + `RestartSec=3` | restart after a crash, wait 3 seconds between attempts |
| `chmod -R o-rwx` | others get no access; the group (`peoplepulse`) can read, root owns and writes |
| `enable --now` | start at boot <b>and</b> start now |

<h4>Teardown Commands</h4>


```bash
# Stop and remove the assessment service (the code stays in Git)
sudo systemctl disable --now peoplepulse
sudo rm -f /etc/systemd/system/peoplepulse.service
sudo systemctl daemon-reload && sudo systemctl reset-failed
sudo rm -rf /opt/peoplepulse

# Stop dev servers and clean local artifacts
pkill -f "uvicorn app.main:app" || true
pkill -f "uvicorn main:app" || true
rm -rf ~/day05-demo
cd ~/devops-8wk/devops-8wk-<your-name> && rm -rf .pytest_cache .coverage htmlcov .ruff_cache
ss -ltn | grep ':8000' || echo "port 8000 free"
```

Expected output:

```text
Removed "/etc/systemd/system/multi-user.target.wants/peoplepulse.service".
port 8000 free
```

<h4>Verify Nothing Is Billing</h4>


```bash
aws ec2 describe-instances --region us-east-1 \
  --filters Name=instance-state-name,Values=running,pending,stopped \
  --query 'Reservations[].Instances[].InstanceId' --output text     # expect empty
az resource list --query "length(@)" -o tsv                        # expect 0 (or only NetworkWatcherRG items)
```

<b>Command breakdown:</b> `aws ec2 describe-instances --filters ... --query ... --output text`

| Part | What it means |
|---|---|
| `--filters Name=instance-state-name,Values=running,pending,stopped` | stopped instances still cost money for their disks, so include them |
| `--query 'Reservations[].Instances[].InstanceId'` | flatten the nested response into a list of instance IDs |
| `--output text` | empty output means nothing is there |
| `az resource list --query "length(@)"` | count every resource in the subscription |

<h3>4:45 – 5:00 PM · Commit & Daily Report</h3>


```bash
cd ~/devops-8wk/devops-8wk-<your-name>
cat >> README.md <<'EOF'

## Run PeoplePulse locally
    python3 -m venv .venv && source .venv/bin/activate
    python -m pip install -r requirements-dev.txt
    uvicorn app.main:app --reload --host 127.0.0.1 --port 8000   # docs: /docs
    pytest --cov=app --cov-fail-under=80 && ruff check . && ruff format --check .
EOF
git add README.md app tests day05
git status --short                    # only app/, tests/, day05/, README.md
git commit -m "test(api): cover record lookup, errors and data file validation"
git push
gh pr create --base main --title "feat(api): PeoplePulse CSV records API with tests" \
  --body "Mini Project 1 app part. /health, /records, /records/{id}; coverage <xx>%; ruff clean."
# after the mentor approves and you merge:
git switch main && git pull
git tag -a v0.2.0 -m "PeoplePulse API + tests - week 1"
git push origin v0.2.0
```

<b>Command breakdown:</b> `git tag -a v0.2.0 -m "..." && git push origin v0.2.0`

| Part | What it means |
|---|---|
| `git tag -a v0.2.0` | create an <b>annotated</b> tag (has an author, date and message) on the current commit |
| `-m "..."` | the tag message |
| `git push origin v0.2.0` | tags are not pushed by `git push`; push each one by name |
| `v0.2.0` | MINOR bump from Day 4's `v0.1.0`: a new feature, still initial development (`0.y.z`) |


<h2>3. Cheat Sheet</h2>


| Task | Command |
|---|---|
| Create / activate venv | `python3 -m venv .venv` · `source .venv/bin/activate` |
| Install pinned packages | `python -m pip install -r requirements-dev.txt` |
| Show / outdated | `python -m pip show fastapi` · `python -m pip list --outdated` |
| Run the API (dev) | `uvicorn app.main:app --reload --host 127.0.0.1 --port 8000` |
| API docs | `http://127.0.0.1:8000/docs` |
| Status code only | `curl -s -o /dev/null -w '%{http_code}\n' URL` |
| Run tests | `pytest` · `pytest -k records -x -v` · `pytest --lf` |
| Coverage gate | `pytest --cov=app --cov-report=term-missing --cov-fail-under=80` |
| HTML coverage | `pytest --cov=app --cov-report=html` then open `htmlcov/index.html` |
| Lint / format | `ruff check . --fix` · `ruff format .` · `ruff format --check .` |
| AWS identity (boto3) | `boto3.client("sts").get_caller_identity()` |
| Azure login (SDK) | `DefaultAzureCredential()` + `ResourceManagementClient(cred, sub_id)` |
| systemd logs | `journalctl -u peoplepulse -n 30 --no-pager` |

```text
json.loads(text) / json.dumps(obj, indent=2, sort_keys=True)
yaml.safe_load(f) / yaml.safe_load_all(f)
csv.DictReader(f) / csv.DictWriter(f, fieldnames=[...])
@app.get("/items/{item_id}", response_model=Item)      # path param: 422 if wrong type
raise HTTPException(status_code=404, detail="...")
monkeypatch.setenv("DATA_FILE", str(tmp_path / "x.csv"))
@pytest.mark.parametrize(("x", "expected"), [(1, 200), (99, 404)])
```
