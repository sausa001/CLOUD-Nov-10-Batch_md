<h1 align="left">Day 05 – Python for DevOps + Mini Project 1 Start – Questions & Answers</h1>

<p>
<img src="https://img.shields.io/badge/Week-02-1F3864?style=flat-square" alt="Week: 02">
<img src="https://img.shields.io/badge/Day-05-2E75B6?style=flat-square" alt="Day: 05">
<img src="https://img.shields.io/badge/Cloud-Local-FF9900?style=flat-square" alt="Cloud: Local">
<img src="https://img.shields.io/badge/Class-8%3A00%E2%80%9312%3A15%20EST-2E75B6?style=flat-square" alt="Class: 8:00–12:15 EST">
<img src="https://img.shields.io/badge/Lab-1%3A15%E2%80%935%3A00%20EST-2E7D32?style=flat-square" alt="Lab: 1:15–5:00 EST">
</p>

<p><a href="Day05_Python_FastAPI_and_MiniProject1_Start.md">Notes</a> · <a href="Day05_Commands_and_Setup.md">Commands & Setup</a> · <b>Questions & Answers</b> · <a href="../README.md">Week overview</a> · <a href="../../README.md">Course index</a></p>



Companion to the main notes: [Day05_Python_FastAPI_and_MiniProject1_Start.md](Day05_Python_FastAPI_and_MiniProject1_Start.md). All questions from today in one place: recap, interview Q&A,
live quiz and the answer key. Use it to revise and to prepare for client interviews.


<h2>1. Stand-up Recap Questions (8:00 AM)</h2>


Answer these from memory before class starts – they review the previous day.

1. When do you rebase and when do you merge? What is the golden rule of rebasing?
2. Name four settings you would turn on for `main` branch protection.
3. The current version is `0.1.0`. A PR adds a new endpoint with the commit `feat(api): add records endpoint`. What is the next version?
4. Someone ran `git reset --hard HEAD~2` and "lost" two commits. How do you get them back?
5. A developer pushed an AWS access key to GitHub. What is the very first thing you do?


<h2>2. Interview-style Questions with Model Answers</h2>


<h3>1. "Why do you use virtual environments?"</h3>

<b>Model answer:</b> To isolate each project's dependencies, so versions do not conflict and installs are reproducible. I create one venv per project, install from pinned requirements files, and never commit the venv. In Docker the image itself gives isolation, but I still install from the same pinned requirements file so the laptop, CI and the image match.

<h3>2. "How do you manage Python dependencies for a service?"</h3>

<b>Model answer:</b> Runtime packages pinned with `==` in `requirements.txt`, because that is what goes into the image. Test and lint tools in `requirements-dev.txt`, which includes the runtime file with `-r`. Dependabot or Renovate opens pull requests for upgrades and CI tests them. For bigger projects I use a lock file with hashes, for example with pip-tools or uv.

<h3>3. "Give an example of Python automation you wrote as a DevOps Engineer."</h3>

<b>Model answer:</b> A tag-compliance script with boto3: it pages through EC2 instances, lists resources missing `owner` and `env` tags, and posts a report. It uses the default credential chain, so it runs on my laptop with SSO and in a scheduled pipeline with an OIDC role. It has a dry-run default, an `--apply` flag, exit codes for CI, and unit tests with a mocked AWS client.

<h3>4. "How do you test an API?"</h3>

<b>Model answer:</b> Unit tests for the logic, such as CSV parsing and edge cases like a missing column, and API tests with FastAPI's TestClient for status codes and bodies, including the 404, 422 and 503 paths. I use fixtures with `tmp_path` and `monkeypatch` for test data, measure coverage with pytest-cov, and fail the build below 80%.

<h3>5. "What is the difference between yaml.load and yaml.safe_load?"</h3>

<b>Model answer:</b> `safe_load` only builds plain data types: dicts, lists, strings and numbers. `load` with an unsafe loader can create arbitrary Python objects, which allows code execution from a malicious file. I always use `safe_load`, and I quote versions and yes/no values to avoid YAML type surprises.

<h3>6. "How do your scripts authenticate to AWS or Azure?"</h3>

<b>Model answer:</b> Never with keys in code or Git. Locally through SSO profiles or `az login`; in CI through OIDC federation to an IAM role or an Entra ID workload identity; in the cloud through IAM roles, EKS Pod Identity or IRSA, and Azure managed identities. The SDK's default credential chain picks the right one automatically, so the code does not change between environments.

<h3>7. "Which status codes should an API return, and why does it matter for operations?"</h3>

<b>Model answer:</b> 200 for success, 404 when the resource does not exist, 422 for invalid input, and 503 when a dependency such as the data file or database is unavailable. Unhandled 500s should be rare. It matters because load balancers, retries and alerts depend on the code class: a 503 can be retried and should page someone, while a 404 is the caller's problem.

<h3>8. "What is ruff and why put a linter in CI?"</h3>

<b>Model answer:</b> ruff is a very fast Python linter and formatter. In CI it is the first and cheapest gate: it catches unused imports, undefined names and likely bugs, and `ruff format --check` enforces one style. Reviewers then spend their time on logic instead of formatting.


<h2>3. Live Quiz (10 questions)</h2>


Try every question first, then check the answer key in section 5.

<b>Q1.</b> Which command creates a virtual environment in the folder `.venv`?
- A) `pip venv .venv`
- B) `python3 -m venv .venv`
- C) `virtualenv --global`
- D) `source .venv`

<b>Q2.</b> What does this print? `import csv, io; r = csv.DictReader(io.StringIO("id,name\n7,Carla\n")); print(type(next(r)["id"]).__name__)`
- A) `int`
- B) `str`
- C) `dict`
- D) `float`

<b>Q3.</b> True or false: `yaml.safe_load` is the recommended way to load YAML from files you do not fully trust.
- A) True
- B) False

<b>Q4.</b> In FastAPI, a request to `/records/abc` where the path parameter is typed `int` returns:
- A) 200
- B) 404
- C) 422
- D) 500

<b>Q5.</b> Which file should list `pytest` and `ruff`?
- A) `requirements.txt`
- B) `requirements-dev.txt`
- C) `.gitignore`
- D) `Dockerfile`

<b>Q6.</b> What does `pytest --cov=app --cov-fail-under=80` do when total coverage is 72%?
- A) Prints a warning but exits 0
- B) Exits with a non-zero code
- C) Deletes the failing tests
- D) Skips the coverage step

<b>Q7.</b> Which built-in pytest fixture gives each test its own temporary folder?
- A) `capsys`
- B) `monkeypatch`
- C) `tmp_path`
- D) `request`

<b>Q8.</b> True or false: boto3 needs access keys written in the script to work.
- A) True
- B) False

<b>Q9.</b> What does this print? `print(json.dumps({"b": 1, "a": 2}, sort_keys=True))`
- A) `{"b": 1, "a": 2}`
- B) `{"a": 2, "b": 1}`
- C) `{a: 2, b: 1}`
- D) An error

<b>Q10.</b> Which ruff command fails in CI when code is not formatted, without changing any file?
- A) `ruff format .`
- B) `ruff check . --fix`
- C) `ruff format --check .`
- D) `ruff clean`


<h2>5. Answer Key</h2>


1. B – `python3 -m venv .venv` creates it; then `source .venv/bin/activate`.
2. B – the `csv` module always returns strings; convert types yourself or use a model.
3. A (True) – `safe_load` only builds plain Python data types.
4. C – type validation fails, so FastAPI answers 422 Unprocessable Entity.
5. B – test and lint tools are development-only dependencies.
6. B – coverage below the threshold makes pytest exit with a failure code, so CI turns red.
7. C – `tmp_path` is a `pathlib.Path` to a unique temporary folder per test.
8. B (False) – boto3 uses the default credential chain (env vars, profile or SSO, roles).
9. B – `sort_keys=True` sorts the keys alphabetically.
10. C – `--check` reports and exits non-zero if any file would be reformatted, without changing files.
