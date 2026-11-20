<h1 align="left">Day 04 – Git & GitHub – Questions & Answers</h1>

<p>
<img src="https://img.shields.io/badge/Week-02-1F3864?style=flat-square" alt="Week: 02">
<img src="https://img.shields.io/badge/Day-04-2E75B6?style=flat-square" alt="Day: 04">
<img src="https://img.shields.io/badge/Cloud-GitHub-FF9900?style=flat-square" alt="Cloud: GitHub">
<img src="https://img.shields.io/badge/Class-8%3A00%E2%80%9312%3A15%20EST-2E75B6?style=flat-square" alt="Class: 8:00–12:15 EST">
<img src="https://img.shields.io/badge/Lab-1%3A15%E2%80%935%3A00%20EST-2E7D32?style=flat-square" alt="Lab: 1:15–5:00 EST">
</p>

<p><a href="Day04_Git_and_GitHub.md">Notes</a> · <a href="Day04_Commands_and_Setup.md">Commands & Setup</a> · <b>Questions & Answers</b> · <a href="../README.md">Week overview</a> · <a href="../../README.md">Course index</a></p>



Companion to the main notes: [Day04_Git_and_GitHub.md](Day04_Git_and_GitHub.md). All questions from today in one place: recap, interview Q&A,
live quiz and the answer key. Use it to revise and to prepare for client interviews.


<h2>1. Stand-up Recap Questions (8:00 AM)</h2>


Answer these from memory before class starts – they review the previous day.

1. What three things does `set -euo pipefail` do?
2. How many usable IPs does a `/24` subnet have on AWS, and why not 256?
3. `curl` exits with 7 for one host and 28 for another. What is different?
4. Why must you pass `-servername` to `openssl s_client`?
5. What makes a subnet "public" in AWS?


<h2>2. Interview-style Questions with Model Answers</h2>


<h3>1. "Merge vs rebase – which do you use and when?"</h3>

<b>Model answer:</b> I rebase my own feature branch onto the latest `main` to keep it current and the history clean, then merge it through a PR, usually with squash merge. I never rebase a shared branch, because it rewrites commits other people already have. If I must push my branch after a rebase, I use `--force-with-lease`, not `--force`.

<h3>2. "What branching strategy would you recommend?"</h3>

<b>Model answer:</b> For most cloud services, trunk-based development: short-lived branches, small PRs, strong CI, feature flags for unfinished work, and tags on `main` for releases. GitFlow fits products with scheduled, versioned releases or several versions supported at the same time, such as mobile apps. I look at the release cadence, team size and test automation before deciding.

<h3>3. "How do you enforce code quality and security in GitHub?"</h3>

<b>Model answer:</b> Branch protection or rulesets on `main`: PR required, at least one approval, code-owner review, stale approvals dismissed, required status checks such as lint, tests and security scans, conversation resolution, no force pushes, and rules that apply to admins too. On top of that, secret scanning with push protection and Dependabot alerts.

<h3>4. "Someone ran git reset --hard and lost commits. Can you recover them?"</h3>

<b>Model answer:</b> Yes, if they were committed. `git reflog` lists every position `HEAD` had. I find the commit before the reset and run `git branch rescue <hash>` or `git reset --hard HEAD@{n}`. Uncommitted changes are not in the reflog, which is one more reason to commit small and often.

<h3>5. "A developer pushed an AWS access key to GitHub. What do you do?"</h3>

<b>Model answer:</b> First, deactivate and rotate the key in IAM immediately. I assume it is compromised, because bots scan GitHub within minutes. Then I check CloudTrail for any use of the key. After that I remove it from history with `git filter-repo` (or BFG), force-push with the team's agreement, ask everyone to re-clone, and turn on secret scanning push protection so it cannot happen again.

<h3>6. "Explain semantic versioning."</h3>

<b>Model answer:</b> `MAJOR.MINOR.PATCH`. MAJOR for breaking changes, MINOR for backward-compatible features, PATCH for bug fixes. With conventional commits (`feat`, `fix`, `BREAKING CHANGE`), the next version and the changelog can be calculated automatically in the pipeline.

<h3>7. "What is CODEOWNERS and how do you use it?"</h3>

<b>Model answer:</b> A file that maps paths to required reviewers. For example, the platform team owns `/terraform/`, `/k8s/` and `/.github/workflows/`, so any change there automatically requests their review, and with "Require review from Code Owners" their approval is mandatory. I always list at least two owners per path so the author is never the only possible approver.

<h3>8. "What is the difference between git fetch and git pull?"</h3>

<b>Model answer:</b> `fetch` downloads new commits and updates remote-tracking branches like `origin/main` without touching my working branch. `pull` is `fetch` plus a merge (or rebase with `--rebase`) into my current branch. I often fetch first, look at `git log HEAD..origin/main`, and then decide.


<h2>3. Live Quiz (10 questions)</h2>


Try every question first, then check the answer key in section 5.

<b>Q1.</b> Which Git object stores file <b>contents</b>?
- A) tree
- B) blob
- C) commit
- D) ref

<b>Q2.</b> What does `cat .git/HEAD` print when you are on branch `main`?
- A) a commit hash
- B) `ref: refs/heads/main`
- C) `main`
- D) `origin/main`

<b>Q3.</b> True or false: it is safe to rebase a branch that teammates have already pulled and built on.
- A) True
- B) False

<b>Q4.</b> A PR removes the `/records` endpoint. The current version is `1.3.2`. What is the next version?
- A) `1.3.3`
- B) `1.4.0`
- C) `2.0.0`
- D) `1.3.2-rc.1`

<b>Q5.</b> Which commit message follows Conventional Commits?
- A) `Fixed bug`
- B) `fix(api): return 404 for unknown record id`
- C) `FIX: api`
- D) `update stuff`

<b>Q6.</b> What does `git push --force-with-lease` do that `--force` does not?
- A) pushes tags too
- B) refuses if the remote has commits you have not seen
- C) skips hooks
- D) squashes commits

<b>Q7.</b> True or false: adding `.env` to `.gitignore` removes it from the repository's history.
- A) True
- B) False

<b>Q8.</b> In `.github/CODEOWNERS`, line 1 is `* @<gh-haiman>` and a later line is `/app/ @<gh-cindia>`. Who owns `app/main.py`?
- A) Haiman
- B) Cindia
- C) both must approve
- D) nobody

<b>Q9.</b> What does this output? `git tag -a v0.1.0 -m "x" && git cat-file -t v0.1.0`
- A) `commit`
- B) `tree`
- C) `tag`
- D) `blob`

<b>Q10.</b> Which command finds a commit that disappeared after `git reset --hard`?
- A) `git status`
- B) `git reflog`
- C) `git stash list`
- D) `git diff`


<h2>5. Answer Key</h2>


1. B – blobs hold file contents; trees hold names and folder structure.
2. B – `HEAD` is a symbolic ref to the current branch.
3. B (False) – rebasing rewrites commits, so teammates' history would no longer match.
4. C – removing an endpoint is a breaking change, so MAJOR: `2.0.0`.
5. B – `type(scope): description`, lowercase and imperative.
6. B – it refuses to overwrite remote commits you have not fetched.
7. B (False) – it only stops future tracking; the file is still in history (rotate, then rewrite history).
8. B – the last matching pattern wins, so `/app/` (Cindia) applies.
9. C – an annotated tag is its own object of type `tag`.
10. B – the reflog records every move of `HEAD`, even to commits no branch points at.
