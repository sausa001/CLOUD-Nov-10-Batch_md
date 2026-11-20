<h1 align="left">Day 04 – Git & GitHub – Commands & Setup Steps</h1>

<p>
<img src="https://img.shields.io/badge/Week-02-1F3864?style=flat-square" alt="Week: 02">
<img src="https://img.shields.io/badge/Day-04-2E75B6?style=flat-square" alt="Day: 04">
<img src="https://img.shields.io/badge/Cloud-GitHub-FF9900?style=flat-square" alt="Cloud: GitHub">
<img src="https://img.shields.io/badge/Class-8%3A00%E2%80%9312%3A15%20EST-2E75B6?style=flat-square" alt="Class: 8:00–12:15 EST">
<img src="https://img.shields.io/badge/Lab-1%3A15%E2%80%935%3A00%20EST-2E7D32?style=flat-square" alt="Lab: 1:15–5:00 EST">
</p>

<p><a href="Day04_Git_and_GitHub.md">Notes</a> · <b>Commands & Setup</b> · <a href="Day04_Questions_and_Answers.md">Questions & Answers</a> · <a href="../README.md">Week overview</a> · <a href="../../README.md">Course index</a></p>



Companion to the main notes: [Day04_Git_and_GitHub.md](Day04_Git_and_GitHub.md). Every command used today, in the order you meet it in class and lab,
with the command breakdown tables that explain each part. Run commands in your own terminal and replace
`<your-name>` with your resource name from the Batch Roster.


<h2>1. Installation & Setup Steps</h2>


Run these before or at the start of the lab (installs, logins, cluster start-up, tool configuration).

<b>From:</b> Prerequisites

```bash
# macOS: brew install gh   |   Windows: winget install -e --id GitHub.cli
# Ubuntu / WSL: sudo apt-get install -y gh
gh auth login          # choose GitHub.com -> SSH -> log in with a browser
gh auth status
```


<h2>2. Commands by Session</h2>


<h3>8:15 – 9:45 AM · Concept Session 1</h3>


<h4>1. Why Git Is the Centre of DevOps</h4>


```bash
git --version
git config --global user.name
git config --global init.defaultBranch main      # new repos start on "main"
```

Expected output:

```text
git version 2.43.0
Jobanjit Singh
```

<h4>2. The Three Areas and the Everyday Commands</h4>


```bash
mkdir -p ~/day04-areas && cd ~/day04-areas && git init -q
echo "PeoplePulse" > README.md
git status --short
git add README.md && git status --short
git commit -qm "docs: add readme" && git log --oneline
echo "CSV Records API" >> README.md
git diff
```

Expected output:

```text
?? README.md
A  README.md
3f1c2ab (HEAD -> main) docs: add readme
diff --git a/README.md b/README.md
index 5c3f1e2..a7b9d04 100644
--- a/README.md
+++ b/README.md
@@ -1 +1,2 @@
 PeoplePulse
+CSV Records API
```

<b>Command breakdown:</b> `git status --short`

| Part | What it means |
|---|---|
| `git status` | compare working tree, staging area and last commit |
| `--short` | one line per file with a two-letter code |
| `??` | untracked – Git does not know this file yet |
| `A` (left column) | added to the staging area |
| ` M` (right column) | modified in the working tree, not staged |
| `M ` (left column) | modified and staged |
| `UU` | both sides changed – a merge conflict |

<h4>3. Git Internals – How Git Really Stores Data</h4>


```bash
cd ~/day04-areas
git cat-file -t HEAD                 # type of the object HEAD points to
git cat-file -p HEAD                 # pretty-print the commit object
git cat-file -p 'HEAD^{tree}'        # the root tree of that commit
git count-objects -v | head -2
```

Expected output:

```text
commit
tree 8a6c4f0b3e1d2c7f9a0b1c2d3e4f5a6b7c8d9e0f
author Jobanjit Singh <jobanjit.singh@example.com> 1762872000 -0500
committer Jobanjit Singh <jobanjit.singh@example.com> 1762872000 -0500

docs: add readme
100644 blob 5c3f1e2a9b8c7d6e5f4a3b2c1d0e9f8a7b6c5d4e    README.md
count: 3
size: 12
```

<b>Command breakdown:</b> `git cat-file -p 'HEAD^{tree}'`

| Part | What it means |
|---|---|
| `git cat-file` | low-level ("plumbing") command that reads one object from `.git/objects` |
| `-t` / `-p` | print the object's type / pretty-print its content |
| `HEAD` | the commit you are on now |
| `^{tree}` | "the tree of" – follow the commit to its root tree |
| quotes | stop the shell from treating `{` and `^` specially |
| `100644 blob ... README.md` | mode (normal file), object type, hash, file name |

<h4>4. Refs, Branches and HEAD</h4>


```bash
cd ~/day04-areas
cat .git/HEAD
cat .git/refs/heads/main
git switch -c feature/records -q && cat .git/HEAD
git branch -v
```

Expected output:

```text
ref: refs/heads/main
3f1c2ab9d4e5f60718293a4b5c6d7e8f90a1b2c3
ref: refs/heads/feature/records
* feature/records 3f1c2ab docs: add readme
  main            3f1c2ab docs: add readme
```

<h4>6. Merge vs Rebase</h4>


```bash
git switch feat/records-endpoint
git fetch origin
git rebase origin/main
# if there is a conflict: fix the file, then
#   git add <file> && git rebase --continue      (or: git rebase --abort to undo)
git push --force-with-lease
```

<b>Command breakdown:</b> `git push --force-with-lease`

| Part | What it means |
|---|---|
| `git push` | upload your branch to `origin` |
| `--force` (plain) | overwrite the remote branch even if it has commits you never saw – can delete a teammate's work |
| `--force-with-lease` | overwrite <b>only if</b> the remote branch is still where you last fetched it; otherwise refuse |
| when needed | after a rebase or amend, your local branch has new hashes, so a normal push is rejected |
| never on | `main` or any shared branch – branch protection blocks it anyway |

<h4>7. Merge Conflicts</h4>


```bash
git status --short
git diff --name-only --diff-filter=U        # list conflicted files only
# edit data/employees.csv: keep both people, give Raymundo id 5, delete all markers
git diff --check && git add data/employees.csv
git commit                                  # during a merge (or: git rebase --continue)
grep -c '<<<<<<<' data/employees.csv        # must print 0
```

Expected output:

```text
UU data/employees.csv
data/employees.csv
[main e5f6a7b] Merge branch 'feature/add-raymundo'
0
```

<b>Command breakdown:</b> `git diff --name-only --diff-filter=U`

| Part | What it means |
|---|---|
| `git diff` | show differences |
| `--name-only` | print only file names, not the changes |
| `--diff-filter=U` | only files in state <b>U</b>nmerged – the conflicted ones |
| `git diff --check` (next line) | warn about leftover conflict markers and trailing spaces; non-zero exit if found |

<h3>10:00 – 11:15 AM · Concept Session 2 + Live Demo</h3>


<h4>8. Pull Requests and Code Review</h4>


```bash
git switch -c docs/readme-run-steps
echo "Run locally: see Day 5" >> README.md
git commit -am "docs: add local run steps"
git push -u origin docs/readme-run-steps
gh pr create --base main --title "docs: add local run steps" \
  --body "What: run steps in README. Why: onboarding. Tested: followed the steps on a clean WSL."
gh pr list
gh pr checkout 3                                              # reviewer: test PR 3 locally
gh pr review 3 --request-changes --body "Please add a test for unknown IDs."
gh pr review 3 --approve --body "LGTM - tested locally."
gh pr merge 3 --squash --delete-branch
```

Expected output:

```text
https://github.com/<gh-haiman>/peoplepulse-team/pull/3
Showing 1 of 1 open pull request in <gh-haiman>/peoplepulse-team
#3  docs: add local run steps  docs/readme-run-steps  about 1 minute ago
- Requested changes to pull request #3
- Approved pull request #3
- Squashed and merged pull request #3 (docs: add local run steps)
- Deleted branch docs/readme-run-steps and switched to branch main
```

<b>Command breakdown:</b> `gh pr create --base main --title "..." --body "..."`

| Part | What it means |
|---|---|
| `gh pr create` | open a pull request for the current branch |
| `--base main` | the branch you want to merge <b>into</b> |
| `--title "..."` | PR title – use the conventional-commit style; with squash merge it becomes the commit message on `main` |
| `--body "..."` | description: what, why, how tested |
| `--fill` (other option) | use the commit message(s) as title and body |
| `--draft` (other option) | open as a draft: visible, but not ready for review |

<b>Command breakdown:</b> `gh pr merge 3 --squash --delete-branch`

| Part | What it means |
|---|---|
| `gh pr merge 3` | merge PR number 3 – fails if protection rules are not met |
| `--squash` | combine all the PR's commits into one commit on `main` (`--merge` / `--rebase` are the other buttons) |
| `--delete-branch` | delete the branch on GitHub and locally after merging |

<h4>9. Branch Protection and Rulesets</h4>


```bash
gh api repos/<gh-haiman>/peoplepulse-team/branches/main/protection \
  --jq '{reviews: .required_pull_request_reviews.required_approving_review_count,
         codeowners: .required_pull_request_reviews.require_code_owner_reviews,
         admins: .enforce_admins.enabled}'
```

<b>Command breakdown:</b> `gh api repos/OWNER/REPO/branches/main/protection --jq '...'`

| Part | What it means |
|---|---|
| `gh api` | call the GitHub REST API with your `gh` login |
| `repos/<gh-haiman>/peoplepulse-team` | owner and repository |
| `/branches/main/protection` | the protection settings of branch `main` (GET by default; `-X PUT` changes them) |
| `--jq '...'` | filter the JSON response with jq syntax, built into `gh` |
| result | `404 Branch not protected` means no rule exists yet |

<h4>11. .gitignore – Keeping Junk and Secrets Out</h4>


```bash
touch app.log .env .env.example
git status --short
git check-ignore -v app.log .env
git rm --cached old-debug.log 2>/dev/null || echo "not tracked"
```

Expected output:

```text
?? .env.example
?? .gitignore
.gitignore:11:*.log	app.log
.gitignore:13:.env	.env
not tracked
```

<b>Command breakdown:</b> `git check-ignore -v app.log .env`

| Part | What it means |
|---|---|
| `git check-ignore` | ask Git whether a path is ignored |
| `-v` | verbose: also show <b>which</b> file, line and pattern ignores it |
| output `.gitignore:11:*.log` | ignored by line 11 of `.gitignore`, pattern `*.log` |
| no output | the file is <b>not</b> ignored – it can be committed |
| `git rm --cached file` (next line) | stop tracking a file that was committed before, but keep it on disk |

<h4>12. Tags and Semantic Versioning</h4>


```bash
git tag -a v0.1.0 -m "PeoplePulse v0.1.0 - skeleton and seed data"
git tag --list 'v*'
git show v0.1.0 --no-patch
git push origin v0.1.0                      # tags are NOT pushed by default
gh release create v0.1.0 --title "v0.1.0" --generate-notes
```

Expected output:

```text
v0.1.0
tag v0.1.0
Tagger: Haiman Sher <haiman.sher@example.com>
Date:   Thu Nov 12 14:40:12 2026 -0500

PeoplePulse v0.1.0 - skeleton and seed data

commit 7d2e9a1c4b5f60718293a4b5c6d7e8f90a1b2c3 (HEAD -> main, tag: v0.1.0, origin/main)
https://github.com/<gh-haiman>/peoplepulse-team/releases/tag/v0.1.0
```

<b>Command breakdown:</b> `gh release create v0.1.0 --title "v0.1.0" --generate-notes`

| Part | What it means |
|---|---|
| `gh release create v0.1.0` | create a GitHub Release from tag `v0.1.0` (creates the tag too if it does not exist) |
| `--title "v0.1.0"` | the release name shown on GitHub |
| `--generate-notes` | GitHub writes release notes from the merged PR titles since the last release |
| why it matters | good PR titles (conventional commits) give good release notes for free |

<h4>13. Conventional Commits</h4>


```bash
git log --oneline -5
```

Expected output:

```text
7d2e9a1 docs: changelog for v0.1.0
5b4c3d2 feat(data): add Raymundo Trancoso
4a3b2c1 feat(data): add Carla Rivera
3c2b1a0 docs: describe peoplepulse and its endpoints
1a2b3c4 chore: repo skeleton, codeowners, pr template, seed data
```

<h4>14. Undo and Recovery: restore, reset, revert, reflog, cherry-pick</h4>


```bash
git log --oneline -1
git revert --no-edit HEAD                   # undo the last commit with a new commit
git log --oneline -2
git reflog -3
```

Expected output:

```text
5b4c3d2 (HEAD -> main) feat(data): add Raymundo Trancoso
[main 8e7f6a5] Revert "feat(data): add Raymundo Trancoso"
8e7f6a5 (HEAD -> main) Revert "feat(data): add Raymundo Trancoso"
5b4c3d2 feat(data): add Raymundo Trancoso
8e7f6a5 (HEAD -> main) HEAD@{0}: revert: Revert "feat(data): add Raymundo Trancoso"
5b4c3d2 HEAD@{1}: commit: feat(data): add Raymundo Trancoso
4a3b2c1 HEAD@{2}: commit: feat(data): add Carla Rivera
```

<b>Command breakdown:</b> `git reset --hard HEAD~2`

| Part | What it means |
|---|---|
| `git reset` | move the current branch to another commit |
| `--hard` | also overwrite the staging area <b>and your files</b> – uncommitted work is gone for good |
| `--soft` (other mode) | move the branch only; the changes stay staged |
| `--mixed` (default) | move the branch and unstage; your files keep the changes |
| `HEAD~2` | two commits before the current one (`HEAD~1` = parent) |
| risk | on a pushed branch this rewrites history – teammates' copies no longer match |

<h4>Live Demo: Conflict, Rebase and a Reflog Rescue</h4>


```bash
mkdir -p ~/day04-demo && cd ~/day04-demo && git init -q -b main
mkdir data
cat > data/employees.csv <<'EOF'
id,name,email,department,title,location,hire_date
1,Haiman Sher,haiman.sher@example.com,Platform,DevOps Engineer,Austin,2023-01-09
2,Cindia Fouda,cindia.fouda@example.com,Data,Data Analyst,Chicago,2022-07-18
3,Jobanjit Singh,jobanjit.singh@example.com,Security,Cloud Security Engineer,Seattle,2021-11-01
EOF
git add . && git commit -qm "feat(data): seed employees.csv"
git log --oneline
```

Expected output:

```text
a1b2c3d (HEAD -> main) feat(data): seed employees.csv
```

```bash
git switch -qc feature/add-carla
echo "4,Carla Rivera,carla.rivera@example.com,QA,QA Engineer,Dallas,2024-03-11" >> data/employees.csv
git commit -qam "feat(data): add Carla Rivera"

git switch -q main
git switch -qc feature/add-raymundo
echo "4,Raymundo Trancoso,raymundo.trancoso@example.com,Platform,SRE,Denver,2024-05-20" >> data/employees.csv
git commit -qam "feat(data): add Raymundo Trancoso"

git switch -q main
git merge -q feature/add-carla          # fast-forward, no conflict
git merge feature/add-raymundo
```

Expected output:

```text
Auto-merging data/employees.csv
CONFLICT (content): Merge conflict in data/employees.csv
Automatic merge failed; fix conflicts and then commit the result.
```

<b>Command breakdown:</b> `git switch -qc feature/add-carla`

| Part | What it means |
|---|---|
| `git switch` | move to another branch (the modern replacement for `git checkout <branch>`) |
| `-c` | create the branch first, starting from where you are now |
| `-q` | quiet – no "Switched to a new branch" message |
| `feature/add-carla` | branch name; the `/` is just part of the name, used to group branches |

```bash
git status --short
tail -5 data/employees.csv
```

Expected output:

```text
UU data/employees.csv
<<<<<<< HEAD
4,Carla Rivera,carla.rivera@example.com,QA,QA Engineer,Dallas,2024-03-11
=======
4,Raymundo Trancoso,raymundo.trancoso@example.com,Platform,SRE,Denver,2024-05-20
>>>>>>> feature/add-raymundo
```

```bash
head -4 data/employees.csv > /tmp/emp.csv
echo "4,Carla Rivera,carla.rivera@example.com,QA,QA Engineer,Dallas,2024-03-11" >> /tmp/emp.csv
echo "5,Raymundo Trancoso,raymundo.trancoso@example.com,Platform,SRE,Denver,2024-05-20" >> /tmp/emp.csv
mv /tmp/emp.csv data/employees.csv
git diff --check && git add data/employees.csv
git commit -qm "merge: add Raymundo Trancoso with id 5"
git log --oneline --graph
```

Expected output:

```text
*   e5f6a7b (HEAD -> main) merge: add Raymundo Trancoso with id 5
|\
| * 9c8d7e6 (feature/add-raymundo) feat(data): add Raymundo Trancoso
* | 4d3c2b1 (feature/add-carla) feat(data): add Carla Rivera
|/
* a1b2c3d feat(data): seed employees.csv
```

<b>Command breakdown:</b> `git log --oneline --graph`

| Part | What it means |
|---|---|
| `git log` | show commit history, newest first |
| `--oneline` | one line per commit: short hash and title |
| `--graph` | draw branch and merge lines on the left: `*` is a commit, the slash and bar lines show branches splitting and joining |
| `--all` (useful extra) | include every branch, not only the current one |
| `(HEAD -> main)` | decoration: which refs point at this commit |

```bash
git reset --hard HEAD~2
git log --oneline
```

Expected output:

```text
a1b2c3d (HEAD -> main) feat(data): seed employees.csv
```

```bash
git reflog -5
```

Expected output:

```text
a1b2c3d (HEAD -> main) HEAD@{0}: reset: moving to HEAD~2
e5f6a7b HEAD@{1}: commit (merge): merge: add Raymundo Trancoso with id 5
4d3c2b1 (feature/add-carla) HEAD@{2}: merge feature/add-carla: Fast-forward
a1b2c3d (HEAD -> main) HEAD@{3}: checkout: moving from feature/add-raymundo to main
9c8d7e6 (feature/add-raymundo) HEAD@{4}: commit: feat(data): add Raymundo Trancoso
```

```bash
git reset --hard HEAD@{1}            # or, safer: git branch rescue e5f6a7b
wc -l data/employees.csv
```

Expected output:

```text
HEAD is now at e5f6a7b merge: add Raymundo Trancoso with id 5
6 data/employees.csv
```

<b>Command breakdown:</b> `git reset --hard HEAD@{1}`

| Part | What it means |
|---|---|
| `HEAD@{1}` | "where `HEAD` was one move ago", read from the reflog |
| `HEAD@{0}` | where `HEAD` is now (after the bad reset) |
| `--hard` | move `main` back to that commit and restore the files |
| `git branch rescue e5f6a7b` (alternative) | create a new branch at the lost commit without touching your current branch – safer when unsure |
| note | reflog is <b>local</b> to your clone; a teammate's reflog does not contain your moves |

```bash
git tag -a v0.1.0 -m "PeoplePulse seed data v0.1.0"
git cat-file -t v0.1.0
git describe --tags
```

Expected output:

```text
tag
v0.1.0
```

<h3>1:15 – 2:45 PM · Lab Part 1 – Task Assignment & Guided Build</h3>


<h4>Task 1 – Lead: Create the Team Repo and Add Teammates</h4>


```bash
cd ~/devops-8wk
gh repo create <gh-haiman>/peoplepulse-team --public --clone \
  --description "PeoplePulse CSV Records API - team repo" --add-readme
cd peoplepulse-team
for u in <gh-cindia> <gh-jobanjit> <gh-carla> <gh-raymundo>; do
  gh api -X PUT "repos/<gh-haiman>/peoplepulse-team/collaborators/$u" -f permission=push
done
gh api repos/<gh-haiman>/peoplepulse-team/invitations --jq '.[].invitee.login'
```

Expected output:

```text
https://github.com/<gh-haiman>/peoplepulse-team
Cloning into 'peoplepulse-team'...
<gh-cindia>
<gh-jobanjit>
<gh-carla>
<gh-raymundo>
```

<b>Command breakdown:</b> `gh repo create <gh-haiman>/peoplepulse-team --public --clone --description "..." --add-readme`

| Part | What it means |
|---|---|
| `gh repo create OWNER/NAME` | create a new repository on GitHub |
| `--public` | visible to everyone – needed for free-plan branch protection; never put secrets in it |
| `--clone` | also clone it into a local folder of the same name |
| `--description "..."` | short text shown on the repo page |
| `--add-readme` | create an initial `README.md` commit so `main` exists |

<b>Command breakdown:</b> `gh api -X PUT "repos/OWNER/REPO/collaborators/$u" -f permission=push`

| Part | What it means |
|---|---|
| `-X PUT` | HTTP method PUT = "add or update" this collaborator |
| `collaborators/$u` | the GitHub user to invite |
| `-f permission=push` | send a form field: `push` = write access (can push branches, open and merge PRs) |
| other values | `pull` read only, `maintain`, `admin` – give the least that works |
| result | an email invitation; the person is not a collaborator until they accept |

```bash
cd ~/devops-8wk
git clone git@github.com:<gh-haiman>/peoplepulse-team.git && cd peoplepulse-team
git log --oneline
```

Expected output:

```text
1f0e9d8 (HEAD -> main, origin/main, origin/HEAD) Initial commit
```

<h4>Task 2 – Lead: Initial Skeleton (the Only Direct Push to main)</h4>


```bash
mkdir -p .github data
cat > .gitignore <<'EOF'
__pycache__/
*.py[cod]
.venv/
.pytest_cache/
.coverage
htmlcov/
*.log
*.pid
.env
.env.*
!.env.example
*.pem
*.key
.terraform/
*.tfstate
*.tfstate.*
.DS_Store
.idea/
.vscode/
EOF

cat > .github/CODEOWNERS <<'EOF'
# Default: anyone on the team can approve, so the lead's own PRs can be approved too.
# The LAST matching pattern wins.
*            @<gh-haiman> @<gh-cindia> @<gh-jobanjit> @<gh-carla> @<gh-raymundo>
/data/       @<gh-carla> @<gh-raymundo> @<gh-haiman>
/app/        @<gh-cindia> @<gh-jobanjit>
/tests/      @<gh-carla> @<gh-raymundo>
/.github/    @<gh-haiman> @<gh-jobanjit>
EOF

cat > .github/pull_request_template.md <<'EOF'
## What
(one or two sentences)

## Why
(ticket or reason)

## How tested
- [ ] ran locally
- [ ] tests added or updated

## Checklist
- [ ] PR title follows conventional commits (feat/fix/docs/chore...)
- [ ] no secrets, keys or .env files
EOF

cat > data/employees.csv <<'EOF'
id,name,email,department,title,location,hire_date
1,Haiman Sher,haiman.sher@example.com,Platform,DevOps Engineer,Austin,2023-01-09
2,Cindia Fouda,cindia.fouda@example.com,Data,Data Analyst,Chicago,2022-07-18
3,Jobanjit Singh,jobanjit.singh@example.com,Security,Cloud Security Engineer,Seattle,2021-11-01
EOF

printf '# Changelog\n\n## [Unreleased]\n' > CHANGELOG.md
git add . && git commit -m "chore: repo skeleton, codeowners, pr template, seed data"
git push origin main
```

Expected output:

```text
[main 2a3b4c5] chore: repo skeleton, codeowners, pr template, seed data
 5 files changed, 48 insertions(+)
To github.com:<gh-haiman>/peoplepulse-team.git
   1f0e9d8..2a3b4c5  main -> main
```

<h4>Task 3 – Lead: Protect main</h4>


```bash
cat > /tmp/protection.json <<'EOF'
{
  "required_status_checks": null,
  "enforce_admins": true,
  "required_pull_request_reviews": {
    "dismiss_stale_reviews": true,
    "require_code_owner_reviews": true,
    "required_approving_review_count": 1
  },
  "restrictions": null,
  "required_linear_history": false,
  "allow_force_pushes": false,
  "allow_deletions": false,
  "required_conversation_resolution": true
}
EOF
gh api -X PUT repos/<gh-haiman>/peoplepulse-team/branches/main/protection \
  --input /tmp/protection.json > /dev/null && rm /tmp/protection.json
gh api repos/<gh-haiman>/peoplepulse-team/branches/main/protection \
  --jq '{reviews: .required_pull_request_reviews.required_approving_review_count, codeowners: .required_pull_request_reviews.require_code_owner_reviews, admins: .enforce_admins.enabled}'
```

<b>Command breakdown:</b> the `protection.json` fields

| Part | What it means |
|---|---|
| `"required_status_checks": null` | no CI checks required yet – we add them on Day 9 |
| `"enforce_admins": true` | rules apply to admins and the repo owner too |
| `"dismiss_stale_reviews": true` | a new push removes earlier approvals |
| `"require_code_owner_reviews": true` | an owner from CODEOWNERS must approve |
| `"required_approving_review_count": 1` | at least one approval; the author cannot approve their own PR |
| `"restrictions": null` | no extra limit on who may merge (only allowed in organizations) |
| `"allow_force_pushes": false` / `"allow_deletions": false` | block `push --force` and deleting `main` |
| `"required_conversation_resolution": true` | all review threads must be resolved before merge |
| `--input file` | send the file as the request body |

```bash
git pull
echo "direct" >> README.md && git commit -qam "test: direct push" && git push origin main
```

Expected output:

```text
remote: error: GH006: Protected branch update failed for refs/heads/main.
remote: - Changes must be made through a pull request.
To github.com:<gh-haiman>/peoplepulse-team.git
 ! [remote rejected] main -> main (protected branch hook declined)
error: failed to push some refs to 'github.com:<gh-haiman>/peoplepulse-team.git'
```

```bash
git reset --hard origin/main      # throw away the test commit
```

<h4>Task 4 – Developers: Raise PRs; Reviewers: Request Changes and Approve</h4>


```bash
git switch main && git pull
git switch -c docs/readme-overview
cat >> README.md <<'EOF'

## About
PeoplePulse is a small FastAPI service that serves employee records from `data/employees.csv`.
Endpoints (built on Day 5): `/health`, `/records`, `/records/{id}`.
EOF
git commit -am "docs: describe peoplepulse and its endpoints"
git push -u origin docs/readme-overview
gh pr create --fill --base main
```

```bash
git switch main && git pull
git switch -c docs/contributing
mkdir -p docs
cat > docs/CONTRIBUTING.md <<'EOF'
# Contributing
- Branch names: feat/, fix/, docs/, chore/, ci/ + short-kebab-description
- Commit and PR titles: Conventional Commits, e.g. "feat(api): add /records endpoint"
- Every change goes through a PR with one approval; no direct pushes to main
EOF
git add docs && git commit -m "docs: add contributing guide"
git push -u origin docs/contributing
gh pr create --fill --base main
```

```bash
gh pr list
gh pr diff 1
gh pr review 1 --request-changes --body "Please mention that the data lives in data/employees.csv and add the port (8000)."
```

```bash
git switch docs/readme-overview
echo "The API listens on port 8000. Data file: data/employees.csv (set with DATA_FILE)." >> README.md
git commit -am "docs: add port and data file location"
git push
```

```bash
gh pr review 1 --approve --body "Thanks, LGTM"
gh pr merge 1 --squash --delete-branch
gh pr review 2 --approve --body "Clear and short - LGTM"
gh pr merge 2 --squash --delete-branch
git switch main && git pull && git log --oneline -3
```

Expected output:

```text
8f7e6d5 (HEAD -> main, origin/main) docs: add contributing guide (#2)
6d5c4b3 docs: describe peoplepulse and its endpoints (#1)
2a3b4c5 chore: repo skeleton, codeowners, pr template, seed data
```

<b>Command breakdown:</b> `gh pr review 1 --request-changes --body "..."`

| Part | What it means |
|---|---|
| `gh pr review 1` | submit a review on PR number 1 |
| `--request-changes` | block the merge until the author responds and the reviewer approves |
| `--approve` (later) | approve; counts towards the required approvals |
| `--comment` (other option) | leave feedback without approving or blocking |
| `--body "..."` | the review text – be specific about what to change |
| note | with `dismiss_stale_reviews`, any later push clears an approval, so approve after the final fix |

<h4>Task 5 – Resolve a Merge Conflict in a PR</h4>


```bash
# Cindia
git switch main && git pull
git switch -c feat/add-employee-carla
echo "4,Carla Rivera,carla.rivera@example.com,QA,QA Engineer,Dallas,2024-03-11" >> data/employees.csv
git commit -am "feat(data): add Carla Rivera" && git push -u origin feat/add-employee-carla
gh pr create --fill

# Jobanjit (from the same main, before Cindia's PR merges)
git switch main && git pull
git switch -c feat/add-employee-raymundo
echo "4,Raymundo Trancoso,raymundo.trancoso@example.com,Platform,SRE,Denver,2024-05-20" >> data/employees.csv
git commit -am "feat(data): add Raymundo Trancoso" && git push -u origin feat/add-employee-raymundo
gh pr create --fill
```

```bash
git fetch origin
git rebase origin/main
```

Expected output:

```text
Auto-merging data/employees.csv
CONFLICT (content): Merge conflict in data/employees.csv
error: could not apply 7f3e2d1... feat(data): add Raymundo Trancoso
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
```

```bash
git diff --check
git add data/employees.csv
git rebase --continue          # opens the editor for the message - save and close
git push --force-with-lease
tail -2 data/employees.csv
```

Expected output:

```text
Successfully rebased and updated refs/heads/feat/add-employee-raymundo.
4,Carla Rivera,carla.rivera@example.com,QA,QA Engineer,Dallas,2024-03-11
5,Raymundo Trancoso,raymundo.trancoso@example.com,Platform,SRE,Denver,2024-05-20
```

<b>Command breakdown:</b> `git rebase origin/main` followed by `git rebase --continue`

| Part | What it means |
|---|---|
| `git fetch origin` | download the latest `main` into `origin/main` first |
| `git rebase origin/main` | replay this branch's commits on top of the latest `main` |
| `CONFLICT ... could not apply 7f3e2d1` | Git stopped on that commit; the files contain markers |
| `git add <file>` | mark the conflict in that file as resolved |
| `git rebase --continue` | finish replaying this commit and go on to the next one |
| `git rebase --abort` | give up and return the branch to how it was before |
| `git push --force-with-lease` | the branch now has new hashes; update the PR branch safely |

<h4>Task 6 – Tag v0.1.0 and Recover a Lost Commit</h4>


```bash
git switch main && git pull
git switch -c docs/changelog-0.1.0
cat > CHANGELOG.md <<'EOF'
# Changelog

## [0.1.0] - 2026-11-12
### Added
- Repository skeleton, CODEOWNERS, PR template, branch protection
- Seed data: 5 employees in data/employees.csv
- README overview and contributing guide
EOF
git commit -am "docs: changelog for v0.1.0" && git push -u origin docs/changelog-0.1.0
gh pr create --fill          # Raymundo approves and merges, then:
git switch main && git pull
git tag -a v0.1.0 -m "PeoplePulse v0.1.0 - skeleton and seed data"
git push origin v0.1.0
gh release create v0.1.0 --title "v0.1.0" --generate-notes
gh release view v0.1.0 --json tagName,name --jq '.tagName + " " + .name'
```

Expected output:

```text
https://github.com/<gh-haiman>/peoplepulse-team/releases/tag/v0.1.0
v0.1.0 v0.1.0
```

```bash
git switch -c scratch/reflog-drill
echo "draft notes" > notes.txt && git add notes.txt && git commit -qm "docs: draft notes"
git switch main
git branch -D scratch/reflog-drill        # "oops" - branch and commit gone
git reflog | grep "draft notes"
git branch scratch/reflog-drill <hash>    # use the hash from the line above
git log --oneline -1 scratch/reflog-drill
```

Expected output:

```text
Deleted branch scratch/reflog-drill (was c4d5e6f).
c4d5e6f HEAD@{1}: commit: docs: draft notes
c4d5e6f (scratch/reflog-drill) docs: draft notes
```

<b>Command breakdown:</b> `git branch -D scratch/reflog-drill` and the rescue

| Part | What it means |
|---|---|
| `git branch -D name` | force-delete a local branch, even if its commits are not merged (`-d` refuses in that case) |
| `(was c4d5e6f)` | Git even prints the hash it pointed at – note it |
| `git reflog \| grep "draft notes"` | search the reflog for the lost commit's message |
| `git branch name <hash>` | create a branch that points at that commit again – the work is back |

<h3>2:45 – 4:00 PM · Lab Part 2 – Independent Challenge</h3>


<h4>Challenge 3 – Break/Fix: a Secret Was Committed</h4>


```bash
git switch main && git pull
git switch -c feat/db-config
echo 'DB_PASSWORD=Sup3rS3cret-fake-lab' > db.prod.env
git add db.prod.env && git commit -m "feat: add db config"
echo 'DATABASE_HOST=db.internal' > config.txt && git add config.txt && git commit -m "feat: add config"
```

<h3>4:00 – 4:45 PM · Lab Review with Trainer & Teardown</h3>


<h4>Teardown Commands</h4>


```bash
cd ~/devops-8wk/peoplepulse-team
git switch main && git pull --prune
git branch --merged main | grep -vE '^\*|main|release/' | xargs -r git branch -d
git push origin --delete feat/db-config 2>/dev/null || true
gh pr list --state open
rm -rf ~/day04-demo ~/day04-areas
```

Expected output:

```text
Deleted branch docs/changelog-0.1.0 (was 9a8b7c6).
Deleted branch feat/add-employee-raymundo (was 3e4f5a6).
no open pull requests in <gh-haiman>/peoplepulse-team
```

<b>Command breakdown:</b> `git branch --merged main | grep -vE '^\*|main|release/' | xargs -r git branch -d`

| Part | What it means |
|---|---|
| `git branch --merged main` | list local branches whose commits are all already in `main` |
| `grep -vE '^\*\|main\|release/'` | drop the current branch (`*`), `main` itself and release branches from the list |
| `xargs -r` | run the next command with the remaining names as arguments; `-r` = do nothing if the list is empty |
| `git branch -d` | safe delete – refuses if a branch is not fully merged |
| `git pull --prune` (before) | also remove `origin/...` refs for branches deleted on GitHub |

<h3>4:45 – 5:00 PM · Commit & Daily Report</h3>


```bash
cd ~/devops-8wk/devops-8wk-<your-name>
git switch main && git pull
git switch -c day04-git-github
mkdir -p day04
cat > day04/notes.md <<'EOF'
# Day 04 - Git & GitHub - <your-name>
- Team repo: https://github.com/<gh-haiman>/peoplepulse-team
- PRs I opened: #
- PRs I reviewed: #
- Conflict I resolved or reviewed: PR #
- Reflog recovery: commit <hash> "docs: draft notes"
- Secret incident note: <one paragraph>
- GH006 screenshot: day04/gh006.png
EOF
git add day04
git commit -m "docs(day04): git and github lab notes"
git push -u origin day04-git-github
gh pr create --fill
```


<h2>3. Cheat Sheet</h2>


| Task | Command |
|---|---|
| New branch | `git switch -c feat/x` |
| Stage and commit | `git add -p` · `git commit -m "feat(scope): message"` |
| Push a new branch | `git push -u origin feat/x` |
| Update branch from main | `git fetch origin && git rebase origin/main` |
| Safe force push (own branch only) | `git push --force-with-lease` |
| Merge | `git merge feat/x` (`--no-ff` always creates a merge commit) |
| Conflict helpers | `git status` · `git diff --name-only --diff-filter=U` · `git diff --check` · `git merge --abort` · `git rebase --abort` |
| History | `git log --oneline --graph --all` · `git log -p -S 'text'` |
| Undo last commit, keep changes | `git reset --soft HEAD~1` |
| Undo a pushed commit safely | `git revert <sha>` |
| Recover | `git reflog` · `git branch rescue <sha>` · `git reset --hard HEAD@{n}` |
| Copy one commit | `git cherry-pick <sha>` |
| Park work | `git stash push -m "wip"` · `git stash pop` |
| Tags | `git tag -a v0.1.0 -m "msg"` · `git push origin v0.1.0` · `git describe --tags` |
| Ignore | `git check-ignore -v file` · `git rm --cached file` |
| Internals | `git cat-file -t/-p <object>` · `cat .git/HEAD` |
| PRs with gh | `gh pr create --fill` · `gh pr review N --approve` · `gh pr merge N --squash --delete-branch` |
| Release with gh | `gh release create v0.1.0 --generate-notes` |
| Protection with gh | `gh api repos/OWNER/REPO/branches/main/protection` |

```text
Conventional commits: feat | fix | docs | style | refactor | test | chore | ci | build | perf | feat! / BREAKING CHANGE
SemVer:               MAJOR (breaking) . MINOR (new feature) . PATCH (fix)     0.y.z = initial development
Undo on shared branch: git revert   |   Undo local only: git reset   |   Find lost commits: git reflog
Golden rule:          rebase your private branch, merge public ones
```
