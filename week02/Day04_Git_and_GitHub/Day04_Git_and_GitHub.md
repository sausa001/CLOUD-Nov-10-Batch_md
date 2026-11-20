<h1 align="left">Day 04 – Git & GitHub</h1>

<p>
<img src="https://img.shields.io/badge/Week-02-1F3864?style=flat-square" alt="Week: 02">
<img src="https://img.shields.io/badge/Day-04-2E75B6?style=flat-square" alt="Day: 04">
<img src="https://img.shields.io/badge/Cloud-GitHub-FF9900?style=flat-square" alt="Cloud: GitHub">
<img src="https://img.shields.io/badge/Class-8%3A00%E2%80%9312%3A15%20EST-2E75B6?style=flat-square" alt="Class: 8:00–12:15 EST">
<img src="https://img.shields.io/badge/Lab-1%3A15%E2%80%935%3A00%20EST-2E7D32?style=flat-square" alt="Lab: 1:15–5:00 EST">
</p>

<p><b>Notes</b> · <a href="Day04_Commands_and_Setup.md">Commands & Setup</a> · <a href="Day04_Questions_and_Answers.md">Questions & Answers</a> · <a href="../README.md">Week overview</a> · <a href="../../README.md">Course index</a><br><sub><b>Deliverable:</b> Team repo with protected main + PR history</sub></p>




<h2>Learning Objectives</h2>

By the end of today you can:
- Explain how Git stores data (blobs, trees, commits, refs, `HEAD`) and use that picture to reason about any Git command.
- Choose between trunk-based development and GitFlow for a client team and justify the choice.
- Merge and rebase branches, and resolve a merge conflict calmly and correctly.
- Run a full pull request workflow: branch, push, open a PR, review, request changes, approve and merge.
- Protect `main` with required reviews, CODEOWNERS and "no force push", so nobody can push to it directly.
- Write conventional commit messages, tag a semantic-version release (`v0.1.0`), and keep junk and secrets out of the repo with `.gitignore`.
- Recover a "lost" commit or deleted branch with `git reflog`.


<h2>Prerequisites</h2>

- Completed [Day 03 – Bash Scripting & Networking](../../Week01/Day03_Bash_Scripting_and_Networking/Day03_Bash_Scripting_and_Networking.md): your `healthcheck.sh` is in your training repo and you have an SSH key.
- Git 2.40 or newer with `user.name` and `user.email` set (Day 1). Check with `git --version` and `git config --global --list`.
- A GitHub account with <b>two-factor authentication (2FA)</b> on, and your SSH public key added. `ssh -T git@github.com` must greet you by name.
- GitHub CLI (`gh`) installed and logged in:

```bash
# macOS: brew install gh   |   Windows: winget install -e --id GitHub.cli
# Ubuntu / WSL: sudo apt-get install -y gh
gh auth login          # choose GitHub.com -> SSH -> log in with a browser
gh auth status
```

```text
github.com
  Logged in to github.com account <your-github-user> (keyring)
  - Git operations protocol: ssh
  - Token scopes: 'gist', 'read:org', 'repo'
```

- The trainer has named today's <b>team lead</b> for the shared repo `peoplepulse-team` (in these notes the example lead is Haiman). Everyone sends the lead their GitHub username before 1:15 PM.

> [!TIP]
> <b>Pro tip:</b> In these notes GitHub usernames are written as placeholders such as `<gh-haiman>` or `<gh-carla>`. Replace each one with that person's real GitHub username.



<h2>Batch Roster – CLOUD Nov-10 Batch</h2>


Use <b>your own row</b> everywhere the notes say `<your-name>` (cloud resource names, tags, Git repo, branch owner). Resource names are lowercase with hyphens and no accents, because AWS and Azure reject special characters such as `ó`.

| Trainee | Linux username | Resource name (`<your-name>`) | Training repo |
|---|---|---|---|
| Haiman Sher | `haiman` | `haiman-sher` | `devops-8wk-haiman-sher` |
| Cindia Josephine Fouda | `cindia` | `cindia-fouda` | `devops-8wk-cindia-fouda` |
| Jobanjit Singh | `jobanjit` | `jobanjit-singh` | `devops-8wk-jobanjit-singh` |
| Carla Rivera | `carla` | `carla-rivera` | `devops-8wk-carla-rivera` |
| Raymundo Trancoso López | `raymundo` | `raymundo-trancoso` | `devops-8wk-raymundo-trancoso` |

> [!TIP]
> <b>Pro tip:</b> Example: Carla tags her Azure VM `owner=carla-rivera` and names her AWS key pair `day02-carla-rivera`. The trainer can then see at a glance whose resource is still running at the 4:00 PM review.



<h2>Today's Schedule (EST)</h2>

| Time (EST) | Block | Type | What we do today |
|---|---|---|---|
| 8:00 – 8:15 AM | Stand-up & recap | Class | Recap Day 3 (strict mode, CIDR, DNS, TLS) |
| 8:15 – 9:45 AM | Concept session 1 | Class | Why Git, the three areas, Git internals, refs and HEAD, trunk-based vs GitFlow, merge vs rebase, merge conflicts |
| 9:45 – 10:00 AM | Break | Break | — |
| 10:00 – 11:15 AM | Concept session 2 + live demo | Class | Pull requests and code review, branch protection and rulesets, CODEOWNERS, `.gitignore`, tags and semver, conventional commits, undo and recovery; demo: conflict, rebase and reflog rescue |
| 11:15 AM – 12:15 PM | Q&A, quiz & lab briefing | Class | Interview Q&A, 10-question quiz, lab briefing |
| 12:15 – 1:15 PM | Lunch | Break | — |
| 1:15 – 2:45 PM | Lab part 1 – task assignment & guided build | Lab | Team repo, protect `main`, raise and review PRs, resolve a conflict, tag `v0.1.0`, reflog drill |
| 2:45 – 4:00 PM | Lab part 2 – independent challenge | Lab | Squash a messy branch, hotfix and cherry-pick, leaked-secret break/fix, release `v0.2.0` |
| 4:00 – 4:45 PM | Lab review with trainer & teardown | Lab | Mentor reviews PR history and settings; clean up branches |
| 4:45 – 5:00 PM | Commit & daily report | Lab | Push lab notes to the `day04-git-github` branch |

> [!IMPORTANT]
> <b>How the day runs:</b> the class from <b>8:00 AM to 12:15 PM EST</b> is recorded every day (4 hours of teaching; recording pauses during the 9:45 break). After lunch, <b>1:15 – 5:00 PM EST</b> is 3 hours 45 minutes of hands-on lab: at 1:15 PM the trainer posts the day's lab tasks, you work through them (take short breaks at your own pace), and you report your results to the trainer at the 4:00 PM review.


---


<h2>8:00 – 8:15 AM · Stand-up & Recap</h2>
<img src="https://img.shields.io/badge/Class-8%3A00%20%E2%80%93%208%3A15%20AM-2E75B6?style=flat-square" alt="Class: 8:00 – 8:15 AM">

| Minutes | Activity | Method |
|---|---|---|
| 8:00 – 8:03 | Attendance; everyone runs `gh auth status` | Discussion |
| 8:03 – 8:12 | Five recap questions on Day 3 (below) | Quiz |
| 8:12 – 8:15 | Today's goal: work like a real client team on one shared repo | Lecture |

Recap questions on Day 3:
1. What three things does `set -euo pipefail` do?
2. How many usable IPs does a `/24` subnet have on AWS, and why not 256?
3. `curl` exits with 7 for one host and 28 for another. What is different?
4. Why must you pass `-servername` to `openssl s_client`?
5. What makes a subnet "public" in AWS?

<b>Today's goal:</b> work like a real client team: nobody pushes to `main`, every change goes through a reviewed pull request, releases are tagged with semantic versions, and you can rescue lost work.

---


<h2>8:15 – 9:45 AM · Concept Session 1</h2>
<img src="https://img.shields.io/badge/Class-8%3A15%20%E2%80%93%209%3A45%20AM-2E75B6?style=flat-square" alt="Class: 8:15 – 9:45 AM">

| Minutes | Activity | Method |
|---|---|---|
| 8:15 – 8:23 | Why Git is the centre of DevOps; the big picture | Lecture |
| 8:23 – 8:35 | The three areas and the everyday commands | Live demo |
| 8:35 – 8:50 | Git internals: blobs, trees, commits | Live demo |
| 8:50 – 9:00 | Refs, branches and `HEAD` | Whiteboard |
| 9:00 – 9:13 | Branching strategies: trunk-based vs GitFlow | Whiteboard |
| 9:13 – 9:28 | Merge vs rebase; GitHub merge buttons | Whiteboard |
| 9:28 – 9:42 | Merge conflicts: why they happen and how to resolve them | Live demo |
| 9:42 – 9:45 | Two quick "what does this command change?" checks | Quiz |

<h3>1. Why Git Is the Centre of DevOps</h3>

<b>In simple words:</b> Git is a tool that records every change to a set of files, who made it and why. GitHub is a website that hosts Git repositories and adds the features a team needs: pull requests, reviews, permissions and automation.

> [!NOTE]
> <b>Simple analogy:</b> Git is like the "track changes" and version history of a shared document, but for a whole project folder, and every person has a full copy of the history on their own laptop. GitHub is the shared office where everyone brings their copy so it can be reviewed and merged.


<b>Why a DevOps Engineer cares:</b>
- Almost everything you do later in this course starts with Git: CI runs on push (Day 8–9), GitOps deploys from Git (Day 14), Terraform plans run on pull requests (Day 18–19).
- Auditors (SOC 2, SOX, HIPAA) read the pull request history to see who approved each production change.
- When something breaks, `git log` and `git diff` tell you what changed and when. That is often the fastest way to find the root cause.
- Interviewers expect you to explain merge vs rebase and to show how you recover from mistakes.

<b>How it works:</b>

```mermaid
flowchart LR
  subgraph Laptop["Engineer laptop"]
    WT["Working tree<br/>edit files"] --> LR["Local repo<br/>commits"]
  end
  LR -->|git push| GH["GitHub repo<br/>peoplepulse-team"]
  GH --> PR["Pull request<br/>review + checks"]
  PR -->|merge| Main["main branch<br/>protected"]
  Main --> CI["CI pipeline<br/>Day 8-9"]
  Main --> Tag["Tag v0.1.0<br/>release"]
  CI --> Deploy["Deploy to<br/>AWS or Azure"]
  Main -.->|git pull| LR
  class WT,LR user
  class GH,PR,Main,Tag git
  class CI cicd
  class Deploy aws
  classDef aws fill:#FFE8CC,stroke:#FF9900,stroke-width:2px,color:#232F3E
  classDef azure fill:#DCEEFF,stroke:#0078D4,stroke-width:2px,color:#0B2A4A
  classDef k8s fill:#E3E9FB,stroke:#326CE5,stroke-width:2px,color:#13284F
  classDef git fill:#FDE5DD,stroke:#F05032,stroke-width:2px,color:#4A1408
  classDef cicd fill:#FFF3C4,stroke:#D4A017,stroke-width:2px,color:#4A3800
  classDef user fill:#E8F5E9,stroke:#2E7D32,stroke-width:2px,color:#1B3D1E
  classDef sec fill:#FDE7E7,stroke:#C62828,stroke-width:2px,color:#4A0E0E
  classDef mon fill:#F3E5F5,stroke:#8E24AA,stroke-width:2px,color:#3A0F45
  classDef data fill:#E0F2F1,stroke:#00897B,stroke-width:2px,color:#0B3B36
  classDef linux fill:#F1F1F1,stroke:#424242,stroke-width:2px,color:#212121
  classDef app fill:#E8EAF6,stroke:#3949AB,stroke-width:2px,color:#1A237E
```
<p align="center"><sub><i>Figure: Big picture - Git and GitHub in the delivery flow</i></sub></p>


Reading the diagram: you edit and commit on your laptop, then push a branch to GitHub. A pull request collects reviews and automated checks; only then is the change merged into the protected `main` branch. From `main`, pipelines build and deploy, and releases are marked with tags. Teammates then pull `main` back to their laptops to stay up to date.

| Term | Meaning |
|---|---|
| Repository (repo) | a project folder plus its full history (the hidden `.git/` folder) |
| Commit | a saved snapshot of the project with author, date, message and parent commit(s) |
| Branch | a movable name that points at one commit |
| Remote | another copy of the repo, usually called `origin`, on GitHub |
| Pull request (PR) | a GitHub request to merge one branch into another, with review and discussion |

<b>Example:</b>

```bash
git --version
git config --global user.name
git config --global init.defaultBranch main      # new repos start on "main"
```

```text
git version 2.43.0
Jobanjit Singh
```

> [!NOTE]
> <b>Real-world scenario:</b> Real scenario: a payment API at a US client starts failing at 3 PM. The on-call engineer runs `git log --since="6 hours ago" --oneline` on the config repo, finds a timeout changed from 30 to 3 seconds in a merged PR, and reverts it in five minutes. During an incident, Git history lets you go back and see exactly what changed.


<h3>2. The Three Areas and the Everyday Commands</h3>

<b>In simple words:</b> Your changes move through three places on your laptop: the <b>working tree</b> (the files you edit), the <b>staging area</b> (also called the index: the changes you have chosen for the next commit) and the <b>local repository</b> (saved commits). `git push` copies commits to the <b>remote</b> on GitHub.

> [!NOTE]
> <b>Simple analogy:</b> Think of packing a parcel. The working tree is your desk with everything on it. The staging area is the open box: you choose what goes in. A commit is sealing and labelling the box. Push is handing it to the courier.


<b>Why a DevOps Engineer cares:</b>
- Staging lets you commit only the files you mean to, and keeps out your `.env` or a 2 GB log.
- Small commits that do one thing are easier to review and easier to revert.
- Once you know where a change "lives", you know which command undoes it.

<b>How it works:</b>

```mermaid
flowchart LR
  W["Working tree<br/>files you edit"] -->|git add| S["Staging area<br/>next commit"]
  S -->|git commit| R["Local repo<br/>commit history"]
  R -->|git push| O["origin<br/>GitHub"]
  O -->|"git fetch"| T["origin/main<br/>tracking branch"]
  T -->|"git merge or rebase"| R
  R -->|"git switch, restore"| W
  class W user
  class S,R,T git
  class O git
  classDef aws fill:#FFE8CC,stroke:#FF9900,stroke-width:2px,color:#232F3E
  classDef azure fill:#DCEEFF,stroke:#0078D4,stroke-width:2px,color:#0B2A4A
  classDef k8s fill:#E3E9FB,stroke:#326CE5,stroke-width:2px,color:#13284F
  classDef git fill:#FDE5DD,stroke:#F05032,stroke-width:2px,color:#4A1408
  classDef cicd fill:#FFF3C4,stroke:#D4A017,stroke-width:2px,color:#4A3800
  classDef user fill:#E8F5E9,stroke:#2E7D32,stroke-width:2px,color:#1B3D1E
  classDef sec fill:#FDE7E7,stroke:#C62828,stroke-width:2px,color:#4A0E0E
  classDef mon fill:#F3E5F5,stroke:#8E24AA,stroke-width:2px,color:#3A0F45
  classDef data fill:#E0F2F1,stroke:#00897B,stroke-width:2px,color:#0B3B36
  classDef linux fill:#F1F1F1,stroke:#424242,stroke-width:2px,color:#212121
  classDef app fill:#E8EAF6,stroke:#3949AB,stroke-width:2px,color:#1A237E
```
<p align="center"><sub><i>Figure: The three local areas and the remote</i></sub></p>


Reading the diagram: `git add` moves a change from your files into the staging area, `git commit` saves the staged snapshot, and `git push` sends commits to GitHub. In the other direction, `git fetch` downloads new commits into `origin/main` without touching your work, and a merge or rebase brings them into your branch. `git pull` is just fetch followed by merge.

| Command | What it does |
|---|---|
| `git status` | show which files are changed, staged or untracked |
| `git add file` / `git add -p` | stage a file / stage chosen pieces of a file |
| `git commit -m "msg"` | save the staged snapshot with a message |
| `git log --oneline --graph` | short history with branch lines |
| `git diff` / `git diff --staged` | unstaged changes / staged changes |
| `git switch -c name` | create a new branch and move to it |
| `git restore file` | throw away unstaged edits to a file |
| `git fetch` / `git pull` | download only / download and merge |
| `git push -u origin name` | upload a branch and remember the link (`-u` = upstream) |

<b>Example:</b>

```bash
mkdir -p ~/day04-areas && cd ~/day04-areas && git init -q
echo "PeoplePulse" > README.md
git status --short
git add README.md && git status --short
git commit -qm "docs: add readme" && git log --oneline
echo "CSV Records API" >> README.md
git diff
```

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

> [!TIP]
> <b>Pro tip:</b> `git add -p` shows each changed block and asks "stage this hunk?". It is the single best habit for clean commits: you see exactly what goes into the commit, and stray debug `print` lines stay out.


<h3>3. Git Internals – How Git Really Stores Data</h3>

<b>In simple words:</b> Inside `.git/objects/`, Git stores everything as <b>objects</b> named by a hash (a fingerprint) of their content. A <b>blob</b> holds a file's contents, a <b>tree</b> holds a folder listing, and a <b>commit</b> points to one tree plus its parent commit(s).

> [!NOTE]
> <b>Simple analogy:</b> A commit is a photo album page. The tree is the page layout (which photo goes where, with captions = file names). The blobs are the photos themselves. If two pages use the same photo, Git stores it only once, because the same content has the same fingerprint.


<b>Why a DevOps Engineer cares:</b>
- Commits are never edited in place; Git only makes new ones. Once you see that, recovery with `reflog` makes sense.
- The commit hash identifies exactly what was deployed, and CI uses it to tag images (`peoplepulse:3f1c2ab`).
- It explains why a file committed once stays in history even after you delete it. That matters a lot when a secret leaks.

<b>How it works:</b>

| Object | Stores | Points to |
|---|---|---|
| blob | file contents only (no name) | — |
| tree | a folder: names and modes, each pointing to a blob or tree | blobs, trees |
| commit | author, committer, date, message, one tree, parent commit(s) | tree, parents |
| tag (annotated) | tagger, date, message, target | usually a commit |

```mermaid
flowchart LR
  C2["commit 9c8d7e6<br/>feat data"] -->|parent| C1["commit 3f1c2ab<br/>docs readme"]
  C2 --> T2["tree<br/>root folder"]
  C1 --> T1["tree<br/>root folder"]
  T2 --> B1["blob<br/>README.md"]
  T1 --> B1
  T2 --> T3["tree<br/>data/"]
  T3 --> B2["blob<br/>employees.csv"]
  class C1,C2 git
  class T1,T2,T3 linux
  class B1,B2 data
  classDef aws fill:#FFE8CC,stroke:#FF9900,stroke-width:2px,color:#232F3E
  classDef azure fill:#DCEEFF,stroke:#0078D4,stroke-width:2px,color:#0B2A4A
  classDef k8s fill:#E3E9FB,stroke:#326CE5,stroke-width:2px,color:#13284F
  classDef git fill:#FDE5DD,stroke:#F05032,stroke-width:2px,color:#4A1408
  classDef cicd fill:#FFF3C4,stroke:#D4A017,stroke-width:2px,color:#4A3800
  classDef user fill:#E8F5E9,stroke:#2E7D32,stroke-width:2px,color:#1B3D1E
  classDef sec fill:#FDE7E7,stroke:#C62828,stroke-width:2px,color:#4A0E0E
  classDef mon fill:#F3E5F5,stroke:#8E24AA,stroke-width:2px,color:#3A0F45
  classDef data fill:#E0F2F1,stroke:#00897B,stroke-width:2px,color:#0B3B36
  classDef linux fill:#F1F1F1,stroke:#424242,stroke-width:2px,color:#212121
  classDef app fill:#E8EAF6,stroke:#3949AB,stroke-width:2px,color:#1A237E
```
<p align="center"><sub><i>Figure: How a commit points to trees and blobs</i></sub></p>


Reading the diagram: the newer commit points to its parent and to a tree for the root folder. That tree lists `README.md` (a blob) and the `data/` folder (another tree with the CSV blob). The `README.md` blob did not change, so both commits share the same blob and Git stores it only once.

<b>Example:</b>

```bash
cd ~/day04-areas
git cat-file -t HEAD                 # type of the object HEAD points to
git cat-file -p HEAD                 # pretty-print the commit object
git cat-file -p 'HEAD^{tree}'        # the root tree of that commit
git count-objects -v | head -2
```

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

> [!IMPORTANT]
> <b>Key point:</b> Remember for interviews: a commit is an immutable snapshot that points to a tree and its parents; a branch is just a pointer to a commit. Rewriting history (rebase, amend) creates <b>new</b> commits. The old ones still exist until garbage collection removes them.


<h3>4. Refs, Branches and HEAD</h3>

<b>In simple words:</b> A <b>ref</b> is a name for a commit. A <b>branch</b> is a ref that moves forward each time you commit. <b>`HEAD`</b> is a special pointer that says which branch (or commit) you are on right now.

> [!NOTE]
> <b>Simple analogy:</b> Commits are pages in a book. A branch is a bookmark. When you write a new page on that branch, the bookmark moves to the new page. `HEAD` is your finger, showing which bookmark you are reading from.


<b>Why a DevOps Engineer cares:</b>
- A branch is just a tiny file, so creating one costs almost nothing. Use one for every change.
- "Detached HEAD" (pointing at a commit, not a branch) confuses many engineers in CI checkouts. After today you will recognise it.
- Pipelines often build "the commit that `main` points to now"; once you understand refs, those pipeline logs are much easier to read.

<b>How it works:</b> refs are small files: `.git/refs/heads/main` contains a commit hash. `.git/HEAD` normally contains `ref: refs/heads/main`.

```mermaid
flowchart RL
  HEAD["HEAD"] --> Main["main"]
  Main --> C3["commit C3"]
  Feat["feature/records"] --> C4["commit C4"]
  Tag["tag v0.1.0"] --> C2["commit C2"]
  C4 -->|parent| C3
  C3 -->|parent| C2
  C2 -->|parent| C1["commit C1"]
  class HEAD user
  class Main,Feat,Tag git
  class C1,C2,C3,C4 data
  classDef aws fill:#FFE8CC,stroke:#FF9900,stroke-width:2px,color:#232F3E
  classDef azure fill:#DCEEFF,stroke:#0078D4,stroke-width:2px,color:#0B2A4A
  classDef k8s fill:#E3E9FB,stroke:#326CE5,stroke-width:2px,color:#13284F
  classDef git fill:#FDE5DD,stroke:#F05032,stroke-width:2px,color:#4A1408
  classDef cicd fill:#FFF3C4,stroke:#D4A017,stroke-width:2px,color:#4A3800
  classDef user fill:#E8F5E9,stroke:#2E7D32,stroke-width:2px,color:#1B3D1E
  classDef sec fill:#FDE7E7,stroke:#C62828,stroke-width:2px,color:#4A0E0E
  classDef mon fill:#F3E5F5,stroke:#8E24AA,stroke-width:2px,color:#3A0F45
  classDef data fill:#E0F2F1,stroke:#00897B,stroke-width:2px,color:#0B3B36
  classDef linux fill:#F1F1F1,stroke:#424242,stroke-width:2px,color:#212121
  classDef app fill:#E8EAF6,stroke:#3949AB,stroke-width:2px,color:#1A237E
```
<p align="center"><sub><i>Figure: HEAD, branches and tags are pointers to commits</i></sub></p>


Reading the diagram: `HEAD` points to the `main` branch, which points to commit C3. The feature branch points to C4, built on top of C3. The tag `v0.1.0` is fixed on C2 forever, while branches move. Each commit points back to its parent, so the whole history is a chain.

<b>Example:</b>

```bash
cd ~/day04-areas
cat .git/HEAD
cat .git/refs/heads/main
git switch -c feature/records -q && cat .git/HEAD
git branch -v
```

```text
ref: refs/heads/main
3f1c2ab9d4e5f60718293a4b5c6d7e8f90a1b2c3
ref: refs/heads/feature/records
* feature/records 3f1c2ab docs: add readme
  main            3f1c2ab docs: add readme
```

> [!WARNING]
> <b>Common mistake:</b> "You are in 'detached HEAD' state" means you checked out a commit or tag, not a branch. New commits here belong to no branch and are easy to lose. If you want to keep work, run `git switch -c my-fix` straight away.


<h3>5. Branching Strategies: Trunk-Based vs GitFlow</h3>

<b>In simple words:</b> A branching strategy is the team's agreement on which branches exist and how changes flow into a release. The two you must know are <b>trunk-based development</b> (everyone merges small changes into `main` often) and <b>GitFlow</b> (long-lived `develop` and `main`, plus feature, release and hotfix branches).

> [!NOTE]
> <b>Simple analogy:</b> Trunk-based is a busy kitchen where every cook adds their ingredient to the one big pot every few minutes and tastes it straight away. GitFlow is several kitchens preparing separate dishes for days, then combining them all at a big banquet. That takes more planning and brings more surprises at the end.


<b>Why a DevOps Engineer cares:</b>
- Your CI/CD pipeline design depends on the strategy: which branches build, which deploy, where tags come from.
- DORA research links short-lived branches and frequent integration with higher delivery performance.
- Clients will ask which one you recommend, and they will want a reason.

<b>How it works:</b>

```mermaid
flowchart LR
  M1["main"] --> M2["main"] --> M3["main"] --> M4["main"] --> M5["main<br/>tag v1.4.0"]
  M1 -.-> F1["feat branch<br/>hours"]
  F1 -->|PR| M2
  M2 -.-> F2["fix branch<br/>1 day"]
  F2 -->|PR| M3
  M3 -.-> F3["feat branch<br/>flag off"]
  F3 -->|PR| M4
  class M1,M2,M3,M4,M5 git
  class F1,F2,F3 user
  classDef aws fill:#FFE8CC,stroke:#FF9900,stroke-width:2px,color:#232F3E
  classDef azure fill:#DCEEFF,stroke:#0078D4,stroke-width:2px,color:#0B2A4A
  classDef k8s fill:#E3E9FB,stroke:#326CE5,stroke-width:2px,color:#13284F
  classDef git fill:#FDE5DD,stroke:#F05032,stroke-width:2px,color:#4A1408
  classDef cicd fill:#FFF3C4,stroke:#D4A017,stroke-width:2px,color:#4A3800
  classDef user fill:#E8F5E9,stroke:#2E7D32,stroke-width:2px,color:#1B3D1E
  classDef sec fill:#FDE7E7,stroke:#C62828,stroke-width:2px,color:#4A0E0E
  classDef mon fill:#F3E5F5,stroke:#8E24AA,stroke-width:2px,color:#3A0F45
  classDef data fill:#E0F2F1,stroke:#00897B,stroke-width:2px,color:#0B3B36
  classDef linux fill:#F1F1F1,stroke:#424242,stroke-width:2px,color:#212121
  classDef app fill:#E8EAF6,stroke:#3949AB,stroke-width:2px,color:#1A237E
```
<p align="center"><sub><i>Figure: Trunk-based development - short branches into main</i></sub></p>


Reading the diagram: each change lives on a short branch for hours or a day, then merges into `main` through a PR. Unfinished features are merged too, but hidden behind a <b>feature flag</b> (a setting that switches the feature on or off). Releases are tags on `main`, which is always deployable.

```mermaid
flowchart LR
  D1["develop"] --> D2["develop"] --> D3["develop"] --> D4["develop"]
  D1 -.-> FB["feature/x<br/>days-weeks"]
  FB --> D3
  D3 -.-> RB["release/1.1<br/>stabilise"]
  RB --> MA["main<br/>tag v1.1.0"]
  RB --> D4
  MA -.-> HF["hotfix/1.1.1"]
  HF --> MB["main<br/>tag v1.1.1"]
  HF --> D4
  class D1,D2,D3,D4 git
  class FB user
  class RB,HF cicd
  class MA,MB git
  classDef aws fill:#FFE8CC,stroke:#FF9900,stroke-width:2px,color:#232F3E
  classDef azure fill:#DCEEFF,stroke:#0078D4,stroke-width:2px,color:#0B2A4A
  classDef k8s fill:#E3E9FB,stroke:#326CE5,stroke-width:2px,color:#13284F
  classDef git fill:#FDE5DD,stroke:#F05032,stroke-width:2px,color:#4A1408
  classDef cicd fill:#FFF3C4,stroke:#D4A017,stroke-width:2px,color:#4A3800
  classDef user fill:#E8F5E9,stroke:#2E7D32,stroke-width:2px,color:#1B3D1E
  classDef sec fill:#FDE7E7,stroke:#C62828,stroke-width:2px,color:#4A0E0E
  classDef mon fill:#F3E5F5,stroke:#8E24AA,stroke-width:2px,color:#3A0F45
  classDef data fill:#E0F2F1,stroke:#00897B,stroke-width:2px,color:#0B3B36
  classDef linux fill:#F1F1F1,stroke:#424242,stroke-width:2px,color:#212121
  classDef app fill:#E8EAF6,stroke:#3949AB,stroke-width:2px,color:#1A237E
```
<p align="center"><sub><i>Figure: GitFlow - long-lived develop and main branches</i></sub></p>


Reading the diagram: features are built on long branches off `develop`. When enough is ready, a `release/` branch is cut to stabilise and test, then merged into `main` (tagged) and back into `develop`. Urgent fixes start from `main` on a `hotfix/` branch and are merged into both. More branches mean more merges, and more merges mean more conflicts.

| | Trunk-based | GitFlow |
|---|---|---|
| Branch lifetime | hours to 2 days | days to weeks |
| Release cadence | continuous, often many per day | scheduled (for example, monthly) |
| Merge pain | low – small diffs | high – big merges |
| Needs | strong CI, feature flags, good tests | release manager, more process |
| Good fit | SaaS, microservices, cloud APIs | versioned products, mobile apps, several supported versions |
| This course | <b>Yes</b> – PeoplePulse uses it | know it for interviews |

<b>Example:</b> branch names the PeoplePulse team will use:

```text
feat/records-endpoint        fix/csv-header-check        docs/readme-run-steps
chore/bump-fastapi           ci/add-shellcheck           hotfix/1.0.1-email-typo
```

> [!WARNING]
> <b>Common mistake:</b> Keeping a feature branch open for weeks "to be safe" is the opposite of safe. After two weeks the merge is huge, the conflicts are painful and the risk is higher. Small PRs merged every day carry far less risk.


<h3>6. Merge vs Rebase</h3>

<b>In simple words:</b> Both bring work from one branch into another. <b>Merge</b> joins the two histories and adds a <b>merge commit</b>. <b>Rebase</b> replays your commits on top of the other branch, creating new copies, so the history looks like one straight line.

> [!NOTE]
> <b>Simple analogy:</b> Merge is two roads joining at a junction: you can always see that there were two roads. Rebase is picking up your road and re-laying it so it starts at the end of the other road. The result is neat and straight, but it is a new road.


<b>Why a DevOps Engineer cares:</b>
- Update your branch from `main` before review, so CI does not surprise you later.
- Rebasing a shared branch rewrites commits others already have. It is a classic way to break a teammate's work.
- Which GitHub merge button you choose decides what the history looks like for every pipeline that reads it.

<b>How it works:</b>

```mermaid
flowchart LR
  subgraph Before["Before"]
    A1["A"] --> B1["B"] --> C1["C main"]
    B1 --> D1["D"] --> E1["E feature"]
  end
  subgraph Merge["git merge feature"]
    A2["A"] --> B2["B"] --> C2["C"] --> M2["M merge commit"]
    B2 --> D2["D"] --> E2["E"] --> M2
  end
  subgraph Rebase["git rebase main"]
    A3["A"] --> B3["B"] --> C3["C"] --> D3["D copy"] --> E3["E copy"]
  end
  class A1,B1,C1,A2,B2,C2,A3,B3,C3 git
  class D1,E1,D2,E2 user
  class M2 cicd
  class D3,E3 app
  classDef aws fill:#FFE8CC,stroke:#FF9900,stroke-width:2px,color:#232F3E
  classDef azure fill:#DCEEFF,stroke:#0078D4,stroke-width:2px,color:#0B2A4A
  classDef k8s fill:#E3E9FB,stroke:#326CE5,stroke-width:2px,color:#13284F
  classDef git fill:#FDE5DD,stroke:#F05032,stroke-width:2px,color:#4A1408
  classDef cicd fill:#FFF3C4,stroke:#D4A017,stroke-width:2px,color:#4A3800
  classDef user fill:#E8F5E9,stroke:#2E7D32,stroke-width:2px,color:#1B3D1E
  classDef sec fill:#FDE7E7,stroke:#C62828,stroke-width:2px,color:#4A0E0E
  classDef mon fill:#F3E5F5,stroke:#8E24AA,stroke-width:2px,color:#3A0F45
  classDef data fill:#E0F2F1,stroke:#00897B,stroke-width:2px,color:#0B3B36
  classDef linux fill:#F1F1F1,stroke:#424242,stroke-width:2px,color:#212121
  classDef app fill:#E8EAF6,stroke:#3949AB,stroke-width:2px,color:#1A237E
```
<p align="center"><sub><i>Figure: Merge versus rebase of the same feature branch</i></sub></p>


Reading the diagram: before, `main` has C and the feature branch has D and E, both starting from B. A merge keeps D and E as they are and ties them in with a new merge commit M. A rebase creates new copies of D and E on top of C. You get a straight line, but with new commit hashes.

| | `git merge` | `git rebase` |
|---|---|---|
| History | true history, adds a merge commit | linear, rewrites your commits |
| Safe on shared branches? | yes | <b>no</b> – never rebase commits others have pulled |
| Conflicts | resolved once | may need resolving once per replayed commit |
| Typical use | merging a PR into `main` | updating <b>your</b> feature branch with the latest `main` |

<b>GitHub merge buttons:</b>

| Button | Result on `main` | When |
|---|---|---|
| Create a merge commit | all branch commits plus a merge commit | you want the full branch history |
| Squash and merge | one commit per PR | most common with trunk-based; clean history |
| Rebase and merge | all commits, linear, no merge commit | each commit is meaningful on its own |

<b>Example:</b>

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

> [!TIP]
> <b>Pro tip:</b> The golden rule: <b>rebase your private branch, merge public ones.</b> If you must rewrite a pushed branch, use `--force-with-lease`, never plain `--force`.


<h3>7. Merge Conflicts</h3>

<b>In simple words:</b> A conflict happens when two branches change the <b>same lines</b> of the same file (or one deletes a file the other edits). Git cannot know which version is right, so it stops and asks you to decide.

> [!NOTE]
> <b>Simple analogy:</b> Two people edit the same sentence in a shared document at the same time. The tool shows both versions side by side and asks a human to write the final sentence.


<b>Why a DevOps Engineer cares:</b>
- Config files (Helm values, Terraform variables, CSV seed data) conflict often because many people touch them.
- A conflict resolved badly can ship broken config that still passes CI (a CSV or YAML file does not "compile").
- Interviewers ask "walk me through resolving a conflict".

<b>How it works:</b> Git writes both versions into the file between markers:

```text
<<<<<<< HEAD
4,Carla Rivera,carla.rivera@example.com,QA,QA Engineer,Dallas,2024-03-11
=======
4,Raymundo Trancoso,raymundo.trancoso@example.com,Platform,SRE,Denver,2024-05-20
>>>>>>> feature/add-raymundo
```

- Between `<<<<<<< HEAD` and `=======`: the version on your current branch.
- Between `=======` and `>>>>>>>`: the version from the branch you are merging in.

```mermaid
flowchart TB
  M["git merge or<br/>git rebase"] --> C{"Conflict?"}
  C -->|no| Done["merge commit<br/>or rebased"]
  C -->|yes| S["git status<br/>shows UU files"]
  S --> E["edit file<br/>remove markers"]
  E --> K["git diff --check<br/>no markers left"]
  K --> A["git add file"]
  A --> F{"merge or rebase?"}
  F -->|merge| CM["git commit"]
  F -->|rebase| RC["git rebase --continue"]
  S -.->|"give up"| AB["merge --abort or<br/>rebase --abort"]
  class M,S,E,K,A git
  class C,F cicd
  class Done,CM,RC app
  class AB sec
  classDef aws fill:#FFE8CC,stroke:#FF9900,stroke-width:2px,color:#232F3E
  classDef azure fill:#DCEEFF,stroke:#0078D4,stroke-width:2px,color:#0B2A4A
  classDef k8s fill:#E3E9FB,stroke:#326CE5,stroke-width:2px,color:#13284F
  classDef git fill:#FDE5DD,stroke:#F05032,stroke-width:2px,color:#4A1408
  classDef cicd fill:#FFF3C4,stroke:#D4A017,stroke-width:2px,color:#4A3800
  classDef user fill:#E8F5E9,stroke:#2E7D32,stroke-width:2px,color:#1B3D1E
  classDef sec fill:#FDE7E7,stroke:#C62828,stroke-width:2px,color:#4A0E0E
  classDef mon fill:#F3E5F5,stroke:#8E24AA,stroke-width:2px,color:#3A0F45
  classDef data fill:#E0F2F1,stroke:#00897B,stroke-width:2px,color:#0B3B36
  classDef linux fill:#F1F1F1,stroke:#424242,stroke-width:2px,color:#212121
  classDef app fill:#E8EAF6,stroke:#3949AB,stroke-width:2px,color:#1A237E
```
<p align="center"><sub><i>Figure: Resolving a merge conflict step by step</i></sub></p>


Reading the diagram: when Git reports a conflict, `git status` lists the files marked `UU`. You edit each one to the correct final content and remove every marker, check with `git diff --check`, then `git add` the file. A merge finishes with `git commit`; a rebase continues with `git rebase --continue`. At any point you can abort and go back to where you started.

<b>Example:</b>

```bash
git status --short
git diff --name-only --diff-filter=U        # list conflicted files only
# edit data/employees.csv: keep both people, give Raymundo id 5, delete all markers
git diff --check && git add data/employees.csv
git commit                                  # during a merge (or: git rebase --continue)
grep -c '<<<<<<<' data/employees.csv        # must print 0
```

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

> [!WARNING]
> <b>Common mistake:</b> A common mistake is committing a file that still contains `<<<<<<<` markers. CI may pass, and production then reads garbage. Always run `git diff --check`, read the final file, and run the tests before you commit the resolution.


> [!TIP]
> <b>Pro tip:</b> VS Code shows "Accept Current", "Accept Incoming" and "Accept Both" buttons above each conflict. They are handy, but "Accept Both" on the CSV above would leave two employees with id 4. Always check that the result makes sense.


---


<h2>9:45 – 10:00 AM · Break</h2>
<img src="https://img.shields.io/badge/Break-9%3A45%20%E2%80%93%2010%3A00%20AM-9E9E9E?style=flat-square" alt="Break: 9:45 – 10:00 AM">

Stretch, refill coffee, and make sure `gh auth status` is green. Both the live demo and the lab need it.

---


<h2>10:00 – 11:15 AM · Concept Session 2 + Live Demo</h2>
<img src="https://img.shields.io/badge/Class-10%3A00%20%E2%80%93%2011%3A15%20AM-2E75B6?style=flat-square" alt="Class: 10:00 – 11:15 AM">

| Minutes | Activity | Method |
|---|---|---|
| 10:00 – 10:10 | Pull requests and code review; the `gh` CLI | Live demo |
| 10:10 – 10:18 | Branch protection and rulesets | Live demo |
| 10:18 – 10:24 | CODEOWNERS | Whiteboard |
| 10:24 – 10:29 | `.gitignore` and keeping secrets out | Live demo |
| 10:29 – 10:36 | Tags and semantic versioning | Whiteboard |
| 10:36 – 10:41 | Conventional commits | Lecture |
| 10:41 – 10:50 | Undo and recovery: restore, reset, revert, reflog, cherry-pick | Whiteboard |
| 10:50 – 11:15 | Live Demo: conflict, rebase and a reflog rescue | Live demo |

<h3>8. Pull Requests and Code Review</h3>

<b>In simple words:</b> A <b>pull request (PR)</b> asks to merge one branch into another, usually into `main`. It shows the changes (the <b>diff</b>), runs automated checks, and collects reviews and comments. On client projects, the PR <b>is</b> the change record, and auditors read it.

> [!NOTE]
> <b>Simple analogy:</b> A PR is like submitting an expense claim. You attach the receipts (the diff), explain what and why (the description), the system checks the totals (CI), and a manager approves or sends it back with questions (review).


<b>Why a DevOps Engineer cares:</b>
- Every infrastructure change you make at a client (Terraform, Helm, pipelines) goes through a PR.
- You review other people's PRs and look for anything that affects security or operations (new ports, new secrets, cost).
- Big PRs and slow reviews increase lead time, one of the DORA metrics.

<b>How it works:</b>

```mermaid
sequenceDiagram
  participant A as Cindia (author)
  participant G as GitHub
  participant CI as CI checks
  participant R as Carla (reviewer)
  A->>G: push branch feat/add-employee
  A->>G: gh pr create
  G->>CI: run checks (from Day 9)
  G->>R: review requested (CODEOWNERS)
  R->>G: request changes - add a test
  A->>G: push fix commit
  CI->>G: checks green
  R->>G: approve
  A->>G: squash and merge, delete branch
```
<p align="center"><sub><i>Figure: Life of a pull request</i></sub></p>


Reading the diagram: Cindia pushes a branch and opens a PR. GitHub starts the checks and automatically asks the code owner (Carla) to review. Carla requests a change, Cindia pushes one more commit to the same branch, the PR updates, checks pass and Carla approves. Only then can the PR be merged.

<b>A good PR:</b>
- Small (aim for fewer than 400 changed lines) and does one thing.
- Title in conventional-commit style: `feat(api): add /records/{id} endpoint`.
- Description: <b>what</b> changed, <b>why</b>, <b>how it was tested</b>, link to the ticket.
- Checks green before you ask for a review.

| Reviewer looks for | Example comment |
|---|---|
| Correctness | "What happens if the CSV is empty? Can we add a test?" |
| Security | "This prints the DB password to the log – please mask it." |
| Readability | "nit: rename `d` to `department`." (prefix small, optional points with <i>nit</i>) |
| Operations impact | "Does this need a new env var in the Helm values?" |
| Cost | "This changes the instance type to `m5.4xlarge` – is that intended?" |

<b>Example:</b>

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

> [!TIP]
> <b>Pro tip:</b> Review the code, not the person: "this function could return `None` when..." instead of "you forgot...". Kind, specific reviews get faster fixes, and US clients notice how you communicate in PRs.


<h3>9. Branch Protection and Rulesets</h3>

<b>In simple words:</b> Branch protection is a set of rules on important branches such as `main`: no direct pushes, a minimum number of approvals, checks that must pass, and no force pushes or deletion. <b>Rulesets</b> are GitHub's newer and more flexible way to set the same rules.

> [!NOTE]
> <b>Simple analogy:</b> Branch protection is the two-signature rule on company cheques: one person cannot write and approve their own big payment, no matter how senior they are.


<b>Why a DevOps Engineer cares:</b>
- It enforces the "four-eyes principle" required by SOC 2 and SOX audits at US clients.
- It stops an accidental `git push --force` on `main` from wiping out history.
- From Day 9, required status checks make sure nothing merges with failing tests.

<b>How it works:</b>

```mermaid
flowchart TB
  P["git push origin main"] -->|rejected GH006| X["Direct push<br/>blocked"]
  PR["Pull request<br/>to main"] --> R1{"1 approval?"}
  R1 -->|no| W["Waiting"]
  R1 -->|yes| R2{"Code owner<br/>approved?"}
  R2 -->|no| W
  R2 -->|yes| R3{"Checks pass and<br/>threads resolved?"}
  R3 -->|no| W
  R3 -->|yes| OK["Merge allowed"]
  class P,PR user
  class X,W sec
  class R1,R2,R3 cicd
  class OK git
  classDef aws fill:#FFE8CC,stroke:#FF9900,stroke-width:2px,color:#232F3E
  classDef azure fill:#DCEEFF,stroke:#0078D4,stroke-width:2px,color:#0B2A4A
  classDef k8s fill:#E3E9FB,stroke:#326CE5,stroke-width:2px,color:#13284F
  classDef git fill:#FDE5DD,stroke:#F05032,stroke-width:2px,color:#4A1408
  classDef cicd fill:#FFF3C4,stroke:#D4A017,stroke-width:2px,color:#4A3800
  classDef user fill:#E8F5E9,stroke:#2E7D32,stroke-width:2px,color:#1B3D1E
  classDef sec fill:#FDE7E7,stroke:#C62828,stroke-width:2px,color:#4A0E0E
  classDef mon fill:#F3E5F5,stroke:#8E24AA,stroke-width:2px,color:#3A0F45
  classDef data fill:#E0F2F1,stroke:#00897B,stroke-width:2px,color:#0B3B36
  classDef linux fill:#F1F1F1,stroke:#424242,stroke-width:2px,color:#212121
  classDef app fill:#E8EAF6,stroke:#3949AB,stroke-width:2px,color:#1A237E
```
<p align="center"><sub><i>Figure: What branch protection checks before a merge</i></sub></p>


Reading the diagram: a direct push to `main` is rejected with error GH006. A pull request can merge only after it has the required approvals, a code owner has approved the files they own, required checks pass, and all review comments are resolved. If any rule fails, the merge button stays disabled.

| Rule | Why |
|---|---|
| Require a pull request before merging | no direct pushes to `main` |
| Required approvals: 1 (or 2) | four-eyes principle |
| Dismiss stale approvals when new commits are pushed | the approval covers the code actually merged |
| Require review from Code Owners | the right experts approve their area |
| Require status checks to pass | CI must be green (we add checks on Day 9) |
| Require conversation resolution | no ignored review comments |
| Block force pushes and deletions | protects history |
| Do not allow bypassing (include administrators) | rules apply to everyone, including the repo owner |

| | Classic branch protection | Rulesets |
|---|---|---|
| Where | Settings → Branches | Settings → Rules → Rulesets |
| Targets | one branch name pattern per rule | many branches <b>and</b> tags in one ruleset |
| Organization-wide | no | yes (on paid plans) |
| Several rules at once | one rule per branch | rulesets stack; the strictest wins |

<b>Example:</b> read the protection on `main` with the API:

```bash
gh api repos/<gh-haiman>/peoplepulse-team/branches/main/protection \
  --jq '{reviews: .required_pull_request_reviews.required_approving_review_count,
         codeowners: .required_pull_request_reviews.require_code_owner_reviews,
         admins: .enforce_admins.enabled}'
```

```json
{"admins":true,"codeowners":true,"reviews":1}
```

<b>Command breakdown:</b> `gh api repos/OWNER/REPO/branches/main/protection --jq '...'`

| Part | What it means |
|---|---|
| `gh api` | call the GitHub REST API with your `gh` login |
| `repos/<gh-haiman>/peoplepulse-team` | owner and repository |
| `/branches/main/protection` | the protection settings of branch `main` (GET by default; `-X PUT` changes them) |
| `--jq '...'` | filter the JSON response with jq syntax, built into `gh` |
| result | `404 Branch not protected` means no rule exists yet |

> [!WARNING]
> <b>Common mistake:</b> On a <b>free personal account</b>, branch protection and rulesets are only enforced on <b>public</b> repositories. Private repos need GitHub Pro, Team or Enterprise. Today's team repo holds no secrets, so make it public.


> [!NOTE]
> <b>Real-world scenario:</b> Real scenario: at a healthcare client, Raymundo is asked in an audit "show me that nobody can deploy to production without review". He shows the ruleset on `main` (PR required, 2 approvals, code owners, admins included) and the PR history. That one screen is all the auditor needs.


<h3>10. CODEOWNERS</h3>

<b>In simple words:</b> `CODEOWNERS` is a file (usually `.github/CODEOWNERS`) that maps folders and files to the people or teams who must review changes to them. GitHub automatically asks them to review, and with "Require review from Code Owners" their approval is required.

> [!NOTE]
> <b>Simple analogy:</b> CODEOWNERS is the list on a building's maintenance board: plumbing problems go to the plumber, electrical to the electrician. Anyone can report a problem, but the right expert must sign off the repair.


<b>Why a DevOps Engineer cares:</b>
- The platform team owns `/terraform/`, `/k8s/`, `/helm/` and `/.github/workflows/`, because changes there can take down production.
- The security team owns IAM policies and secret configuration.
- Nobody has to ask "who should review this?", so PRs move faster.

<b>How it works:</b> each line is a path pattern and one or more owners. <b>The last matching line wins.</b>

```mermaid
flowchart LR
  PR["PR changes<br/>data/employees.csv<br/>and .github/"] --> CO["CODEOWNERS<br/>last match wins"]
  CO -->|"/data/"| O1["Haiman<br/>Cindia"]
  CO -->|"/.github/"| O2["Haiman"]
  CO -->|"/tests/"| O3["Carla<br/>Raymundo"]
  O1 --> Rev["review requests<br/>sent automatically"]
  O2 --> Rev
  class PR git
  class CO sec
  class O1,O2,O3 user
  class Rev git
  classDef aws fill:#FFE8CC,stroke:#FF9900,stroke-width:2px,color:#232F3E
  classDef azure fill:#DCEEFF,stroke:#0078D4,stroke-width:2px,color:#0B2A4A
  classDef k8s fill:#E3E9FB,stroke:#326CE5,stroke-width:2px,color:#13284F
  classDef git fill:#FDE5DD,stroke:#F05032,stroke-width:2px,color:#4A1408
  classDef cicd fill:#FFF3C4,stroke:#D4A017,stroke-width:2px,color:#4A3800
  classDef user fill:#E8F5E9,stroke:#2E7D32,stroke-width:2px,color:#1B3D1E
  classDef sec fill:#FDE7E7,stroke:#C62828,stroke-width:2px,color:#4A0E0E
  classDef mon fill:#F3E5F5,stroke:#8E24AA,stroke-width:2px,color:#3A0F45
  classDef data fill:#E0F2F1,stroke:#00897B,stroke-width:2px,color:#0B3B36
  classDef linux fill:#F1F1F1,stroke:#424242,stroke-width:2px,color:#212121
  classDef app fill:#E8EAF6,stroke:#3949AB,stroke-width:2px,color:#1A237E
```
<p align="center"><sub><i>Figure: CODEOWNERS routes reviews by path</i></sub></p>


Reading the diagram: a PR touches the CSV and a file under `.github/`. GitHub checks each changed file against CODEOWNERS, using the last matching line, and requests reviews from those owners. The `/tests/` owners are not asked because no test file changed.

<b>Example:</b>

```text
# .github/CODEOWNERS - the LAST matching pattern wins
*            @<gh-haiman> @<gh-cindia> @<gh-jobanjit> @<gh-carla> @<gh-raymundo>
/data/       @<gh-haiman> @<gh-cindia>
/app/        @<gh-cindia> @<gh-jobanjit>
/tests/      @<gh-carla> @<gh-raymundo>
/.github/    @<gh-haiman>
```

| Pattern | Matches |
|---|---|
| `*` | every file (the default owner line – put it first) |
| `/data/` | the `data` folder at the repo root and everything inside |
| `*.tf` | every Terraform file anywhere |
| `docs/` (no leading `/`) | any folder named `docs`, at any depth |
| `@org/team-name` | a GitHub team (inside an organization) |

> [!WARNING]
> <b>Common mistake:</b> If the only code owner of a path is the PR author, nobody else can give the required code-owner approval and the PR is stuck. Always list at least two owners per path, as above. Code owners must also have write access to the repo, or GitHub ignores them.


<h3>11. .gitignore – Keeping Junk and Secrets Out</h3>

<b>In simple words:</b> `.gitignore` lists file patterns that Git should never track: virtual environments, build output, logs, local settings and secrets.

> [!NOTE]
> <b>Simple analogy:</b> `.gitignore` is the "do not pack" list stuck to your suitcase: dirty laundry, rubbish and your house keys stay home.


<b>Why a DevOps Engineer cares:</b>
- Committed `.env` files, `.pem` keys and `terraform.tfstate` (which contains secrets in plain text) are among the most common security findings.
- Committed `.venv/` or `node_modules/` folders make repos huge and slow every CI clone.
- When the repo holds only source files, every pipeline build starts from the same clean state.

<b>How it works:</b>

```text
# Python
__pycache__/
*.py[cod]
.venv/
.pytest_cache/
.coverage
htmlcov/
# Logs and runtime files
*.log
*.pid
# Secrets and local config - NEVER commit
.env
.env.*
!.env.example
*.pem
*.key
# Terraform (Week 4)
.terraform/
*.tfstate
*.tfstate.*
# Editors and OS
.vscode/
.idea/
.DS_Store
```

| Pattern | Meaning |
|---|---|
| `*.log` | any file ending in `.log`, in any folder |
| `.venv/` | a folder named `.venv` (trailing `/` = folders only) |
| `*.py[cod]` | `.pyc`, `.pyo` or `.pyd` |
| `.env.*` | `.env.prod`, `.env.local` ... |
| `!.env.example` | `!` = exception: DO track this file (a template with no real values) |

<b>Example:</b>

```bash
touch app.log .env .env.example
git status --short
git check-ignore -v app.log .env
git rm --cached old-debug.log 2>/dev/null || echo "not tracked"
```

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

> [!CAUTION]
> <b>Cost / security alert:</b> Adding `.env` to `.gitignore` <b>after</b> it was committed does not remove it from history. If a secret was ever pushed, treat it as leaked: <b>rotate (replace) it first</b>, then clean the history. Bots scan public GitHub repos and find leaked keys within minutes.


<h3>12. Tags and Semantic Versioning</h3>

<b>In simple words:</b> A <b>tag</b> is a permanent name for one commit, used to mark releases (`v0.1.0`). <b>Semantic Versioning (semver)</b> is the rule for choosing the numbers: `MAJOR.MINOR.PATCH`.

> [!NOTE]
> <b>Simple analogy:</b> A tag is a label on a wine bottle with the vintage year: the wine in that bottle never changes. Semver is the numbering rule the winery promises to follow, so buyers know whether a new bottle is a small tweak or a completely different wine.


<b>Why a DevOps Engineer cares:</b>
- From Week 2, Git tags become Docker image tags (`peoplepulse:0.1.0`) and Helm chart versions.
- A release tag tells you exactly which commit is running in production, so you can roll back to the previous tag.
- A version number tells other teams at a glance whether an upgrade is safe.

<b>How it works:</b>

```mermaid
flowchart TB
  Q1{"Breaking change?<br/>removed or renamed"} -->|yes| MJ["MAJOR<br/>1.4.2 to 2.0.0"]
  Q1 -->|no| Q2{"New feature,<br/>backward compatible?"}
  Q2 -->|yes| MN["MINOR<br/>1.4.2 to 1.5.0"]
  Q2 -->|no| Q3{"Bug fix only?"}
  Q3 -->|yes| PT["PATCH<br/>1.4.2 to 1.4.3"]
  Q3 -->|no| NO["no release<br/>docs, chore"]
  class Q1,Q2,Q3 cicd
  class MJ sec
  class MN app
  class PT git
  class NO linux
  classDef aws fill:#FFE8CC,stroke:#FF9900,stroke-width:2px,color:#232F3E
  classDef azure fill:#DCEEFF,stroke:#0078D4,stroke-width:2px,color:#0B2A4A
  classDef k8s fill:#E3E9FB,stroke:#326CE5,stroke-width:2px,color:#13284F
  classDef git fill:#FDE5DD,stroke:#F05032,stroke-width:2px,color:#4A1408
  classDef cicd fill:#FFF3C4,stroke:#D4A017,stroke-width:2px,color:#4A3800
  classDef user fill:#E8F5E9,stroke:#2E7D32,stroke-width:2px,color:#1B3D1E
  classDef sec fill:#FDE7E7,stroke:#C62828,stroke-width:2px,color:#4A0E0E
  classDef mon fill:#F3E5F5,stroke:#8E24AA,stroke-width:2px,color:#3A0F45
  classDef data fill:#E0F2F1,stroke:#00897B,stroke-width:2px,color:#0B3B36
  classDef linux fill:#F1F1F1,stroke:#424242,stroke-width:2px,color:#212121
  classDef app fill:#E8EAF6,stroke:#3949AB,stroke-width:2px,color:#1A237E
```
<p align="center"><sub><i>Figure: Choosing the next semantic version</i></sub></p>


Reading the diagram: ask the questions from the top. A breaking change (removing an endpoint, renaming a JSON field) bumps MAJOR and resets the others to zero. A new compatible feature bumps MINOR. A fix only bumps PATCH. Changes that only touch documentation or chores do not need a release.

| Version | Meaning |
|---|---|
| `0.y.z` | initial development – anything may change; PeoplePulse starts at `v0.1.0` |
| `1.0.0` | first stable public API |
| `2.0.0-rc.1` | pre-release (release candidate) – sorts before `2.0.0` |

| | Annotated tag | Lightweight tag |
|---|---|---|
| Command | `git tag -a v0.1.0 -m "msg"` | `git tag v0.1.0` |
| Stores | tagger, date, message (its own object) | just a ref to a commit |
| Use for | releases | private bookmarks |

<b>Example:</b>

```bash
git tag -a v0.1.0 -m "PeoplePulse v0.1.0 - skeleton and seed data"
git tag --list 'v*'
git show v0.1.0 --no-patch
git push origin v0.1.0                      # tags are NOT pushed by default
gh release create v0.1.0 --title "v0.1.0" --generate-notes
```

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

> [!WARNING]
> <b>Common mistake:</b> Never move or re-use a release tag (`git tag -f v0.1.0`). Pipelines, image registries and other teams may already depend on it. If `v0.1.0` is broken, release `v0.1.1`.


<h3>13. Conventional Commits</h3>

<b>In simple words:</b> Conventional Commits is a simple format for commit messages: `type(scope): short description`. The type tells everyone (and tools) what kind of change it is.

> [!NOTE]
> <b>Simple analogy:</b> It is like labelling moving boxes "KITCHEN: plates" instead of "stuff". Anyone can find things quickly, and the movers know which boxes are fragile.


<b>Why a DevOps Engineer cares:</b>
- Tools such as release-please and semantic-release read the types to pick the next version and write the changelog automatically.
- During an incident, you can scan `git log --oneline` and quickly spot the change that matters.
- Many client repos enforce the format with a CI check on PR titles.

<b>How it works:</b>

| Type | Use | Semver effect |
|---|---|---|
| `feat` | new feature | MINOR |
| `fix` | bug fix | PATCH |
| `docs`, `style`, `refactor`, `test`, `chore`, `ci`, `build`, `perf` | no user-facing change | none |
| `feat!:` or a `BREAKING CHANGE:` footer | breaking change | MAJOR |

Rules: lowercase type, optional scope in brackets, a colon and a space, then a short description in the imperative mood ("add", not "added"), no full stop at the end.

<b>Example:</b>

```text
feat(api): add /records/{id} endpoint

Returns 404 with {"detail": "Record not found"} for unknown IDs.

Refs: PP-12
```

```bash
git log --oneline -5
```

```text
7d2e9a1 docs: changelog for v0.1.0
5b4c3d2 feat(data): add Raymundo Trancoso
4a3b2c1 feat(data): add Carla Rivera
3c2b1a0 docs: describe peoplepulse and its endpoints
1a2b3c4 chore: repo skeleton, codeowners, pr template, seed data
```

> [!TIP]
> <b>Pro tip:</b> With squash merge, only the <b>PR title</b> lands on `main`. So the PR title is the message that matters most. Write it in conventional-commit style even if your branch commits were messy.


<h3>14. Undo and Recovery: restore, reset, revert, reflog, cherry-pick</h3>

<b>In simple words:</b> Git has a different "undo" for each situation. `restore` throws away file edits, `reset` moves your branch backwards, `revert` adds a new commit that undoes an old one, `reflog` shows everywhere `HEAD` has been (so you can find "lost" commits), and `cherry-pick` copies one commit onto another branch.

> [!NOTE]
> <b>Simple analogy:</b> `revert` is printing a correction in tomorrow's newspaper: yesterday's paper is still out there. `reset` is pulling a draft off your own desk before anyone saw it. `reflog` is the CCTV recording of your desk: even if you threw a page away, you can see where it went.


<b>Why a DevOps Engineer cares:</b>
- A bad change on `main` must be undone with `revert` (safe for shared history), never with `reset` plus force push.
- Hotfixes often need `cherry-pick` onto a release branch.
- "I deleted my branch" or "reset --hard ate my work" happens to everyone sooner or later; reflog gets the work back.

<b>How it works:</b>

| Situation | Command | Safe on shared branches? |
|---|---|---|
| Throw away unstaged edits in a file | `git restore file` | yes (local only) |
| Unstage a file, keep the edits | `git restore --staged file` | yes |
| Undo last commit, keep changes staged | `git reset --soft HEAD~1` | only if not pushed |
| Undo last commit and the changes | `git reset --hard HEAD~1` | only if not pushed – destructive |
| Undo a commit that is already on `main` | `git revert <sha>` | <b>yes</b> – adds a new commit |
| Find a lost commit or deleted branch | `git reflog` then `git branch rescue <sha>` | yes |
| Copy one commit to another branch | `git cherry-pick <sha>` | yes |
| Park work-in-progress for a moment | `git stash push -m "wip"` / `git stash pop` | yes |

```mermaid
flowchart LR
  C1["C1 seed"] --> C2["C2 add Carla"] --> C3["C3 merge"]
  R["git reset --hard HEAD~2"] -->|"main moves back"| C1
  RL["git reflog<br/>HEAD@{1} = C3"] --> FIX["git branch rescue C3<br/>or reset --hard HEAD@{1}"]
  FIX --> C3
  class C1,C2,C3 git
  class R sec
  class RL mon
  class FIX app
  classDef aws fill:#FFE8CC,stroke:#FF9900,stroke-width:2px,color:#232F3E
  classDef azure fill:#DCEEFF,stroke:#0078D4,stroke-width:2px,color:#0B2A4A
  classDef k8s fill:#E3E9FB,stroke:#326CE5,stroke-width:2px,color:#13284F
  classDef git fill:#FDE5DD,stroke:#F05032,stroke-width:2px,color:#4A1408
  classDef cicd fill:#FFF3C4,stroke:#D4A017,stroke-width:2px,color:#4A3800
  classDef user fill:#E8F5E9,stroke:#2E7D32,stroke-width:2px,color:#1B3D1E
  classDef sec fill:#FDE7E7,stroke:#C62828,stroke-width:2px,color:#4A0E0E
  classDef mon fill:#F3E5F5,stroke:#8E24AA,stroke-width:2px,color:#3A0F45
  classDef data fill:#E0F2F1,stroke:#00897B,stroke-width:2px,color:#0B3B36
  classDef linux fill:#F1F1F1,stroke:#424242,stroke-width:2px,color:#212121
  classDef app fill:#E8EAF6,stroke:#3949AB,stroke-width:2px,color:#1A237E
```
<p align="center"><sub><i>Figure: Reflog rescue after a hard reset</i></sub></p>


Reading the diagram: a hard reset moved `main` back to C1, so C2 and C3 disappear from `git log`. But the commits still exist, and the reflog remembers that `HEAD` was at C3 one step ago (`HEAD@{1}`). Pointing a branch at C3 again brings everything back.

<b>Example:</b>

```bash
git log --oneline -1
git revert --no-edit HEAD                   # undo the last commit with a new commit
git log --oneline -2
git reflog -3
```

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

> [!IMPORTANT]
> <b>Key point:</b> Remember for interviews: "On shared branches I undo with `git revert`, which adds a new commit. I use `reset` only on local, unpushed work. Anything that was committed can usually be recovered with `git reflog` for about 90 days; uncommitted changes cannot."


<h3>Live Demo: Conflict, Rebase and a Reflog Rescue</h3>

For the demo the trainer uses a throw-away local repo with the PeoplePulse seed data. Two teammates add a different person with the same ID, the merge conflicts, the trainer resolves it, then "loses" the work and rescues it.

```mermaid
flowchart LR
  S["Seed CSV<br/>3 employees"] --> B1["branch add-carla<br/>id 4"]
  S --> B2["branch add-raymundo<br/>id 4"]
  B1 -->|fast-forward| M["main"]
  B2 -->|merge| CF["CONFLICT<br/>same line"]
  CF --> RS["resolve<br/>Raymundo = id 5"]
  RS --> RH["reset --hard<br/>work lost"]
  RH --> RF["reflog<br/>rescue"]
  RF --> T["tag v0.1.0"]
  class S data
  class B1,B2,M git
  class CF,RH sec
  class RS,RF app
  class T git
  classDef aws fill:#FFE8CC,stroke:#FF9900,stroke-width:2px,color:#232F3E
  classDef azure fill:#DCEEFF,stroke:#0078D4,stroke-width:2px,color:#0B2A4A
  classDef k8s fill:#E3E9FB,stroke:#326CE5,stroke-width:2px,color:#13284F
  classDef git fill:#FDE5DD,stroke:#F05032,stroke-width:2px,color:#4A1408
  classDef cicd fill:#FFF3C4,stroke:#D4A017,stroke-width:2px,color:#4A3800
  classDef user fill:#E8F5E9,stroke:#2E7D32,stroke-width:2px,color:#1B3D1E
  classDef sec fill:#FDE7E7,stroke:#C62828,stroke-width:2px,color:#4A0E0E
  classDef mon fill:#F3E5F5,stroke:#8E24AA,stroke-width:2px,color:#3A0F45
  classDef data fill:#E0F2F1,stroke:#00897B,stroke-width:2px,color:#0B3B36
  classDef linux fill:#F1F1F1,stroke:#424242,stroke-width:2px,color:#212121
  classDef app fill:#E8EAF6,stroke:#3949AB,stroke-width:2px,color:#1A237E
```
<p align="center"><sub><i>Figure: Live demo - from conflict to rescue and tag</i></sub></p>


Reading the diagram: both branches start from the same three-person CSV and each appends a person with id 4. The first merge is a simple fast-forward; the second conflicts on the same line. After resolving it, the trainer simulates a disaster with `reset --hard`, rescues the work from the reflog, and finally tags the result.

1. Create a repo with seed data:

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

```text
a1b2c3d (HEAD -> main) feat(data): seed employees.csv
```

2. Two branches add a different person with the same ID:

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

3. Inspect and resolve: keep both people and give Raymundo ID 5:

```bash
git status --short
tail -5 data/employees.csv
```

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

4. "Disaster": someone runs a hard reset and two commits vanish:

```bash
git reset --hard HEAD~2
git log --oneline
```

```text
a1b2c3d (HEAD -> main) feat(data): seed employees.csv
```

5. Rescue with the reflog. Every move of `HEAD` is recorded locally (kept about 90 days by default):

```bash
git reflog -5
```

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

6. Tag the rescued commit and look at the tag object:

```bash
git tag -a v0.1.0 -m "PeoplePulse seed data v0.1.0"
git cat-file -t v0.1.0
git describe --tags
```

```text
tag
v0.1.0
```

> [!IMPORTANT]
> <b>Key point:</b> Stress this to the class: nothing was really lost, because commits are immutable objects and the reflog remembered them. The same thinking explains why a leaked secret is still in history after you delete the file. You will deal with that in Challenge 3.


---


<h2>11:15 AM – 12:15 PM · Q&A, Live Quiz & Lab Briefing</h2>
<img src="https://img.shields.io/badge/Class-11%3A15%20AM%20%E2%80%93%2012%3A15%20PM-2E75B6?style=flat-square" alt="Class: 11:15 AM – 12:15 PM">

| Minutes | Activity | Method |
|---|---|---|
| 11:15 – 11:35 | Interview-style Q&A (8 questions) | Discussion |
| 11:35 – 11:55 | Live quiz (10 questions) and answer review | Quiz |
| 11:55 – 12:10 | Lab briefing: roles, tasks, acceptance criteria, time-boxes | Lecture |
| 12:10 – 12:15 | Everyone posts their GitHub username in the batch channel for the lead | Hands-on |

<h3>Interview-style Q&A</h3>


<h4>1. "Merge vs rebase – which do you use and when?"</h4>

<b>Model answer:</b> I rebase my own feature branch onto the latest `main` to keep it current and the history clean, then merge it through a PR, usually with squash merge. I never rebase a shared branch, because it rewrites commits other people already have. If I must push my branch after a rebase, I use `--force-with-lease`, not `--force`.

<h4>2. "What branching strategy would you recommend?"</h4>

<b>Model answer:</b> For most cloud services, trunk-based development: short-lived branches, small PRs, strong CI, feature flags for unfinished work, and tags on `main` for releases. GitFlow fits products with scheduled, versioned releases or several versions supported at the same time, such as mobile apps. I look at the release cadence, team size and test automation before deciding.

<h4>3. "How do you enforce code quality and security in GitHub?"</h4>

<b>Model answer:</b> Branch protection or rulesets on `main`: PR required, at least one approval, code-owner review, stale approvals dismissed, required status checks such as lint, tests and security scans, conversation resolution, no force pushes, and rules that apply to admins too. On top of that, secret scanning with push protection and Dependabot alerts.

<h4>4. "Someone ran git reset --hard and lost commits. Can you recover them?"</h4>

<b>Model answer:</b> Yes, if they were committed. `git reflog` lists every position `HEAD` had. I find the commit before the reset and run `git branch rescue <hash>` or `git reset --hard HEAD@{n}`. Uncommitted changes are not in the reflog, which is one more reason to commit small and often.

<h4>5. "A developer pushed an AWS access key to GitHub. What do you do?"</h4>

<b>Model answer:</b> First, deactivate and rotate the key in IAM immediately. I assume it is compromised, because bots scan GitHub within minutes. Then I check CloudTrail for any use of the key. After that I remove it from history with `git filter-repo` (or BFG), force-push with the team's agreement, ask everyone to re-clone, and turn on secret scanning push protection so it cannot happen again.

<h4>6. "Explain semantic versioning."</h4>

<b>Model answer:</b> `MAJOR.MINOR.PATCH`. MAJOR for breaking changes, MINOR for backward-compatible features, PATCH for bug fixes. With conventional commits (`feat`, `fix`, `BREAKING CHANGE`), the next version and the changelog can be calculated automatically in the pipeline.

<h4>7. "What is CODEOWNERS and how do you use it?"</h4>

<b>Model answer:</b> A file that maps paths to required reviewers. For example, the platform team owns `/terraform/`, `/k8s/` and `/.github/workflows/`, so any change there automatically requests their review, and with "Require review from Code Owners" their approval is mandatory. I always list at least two owners per path so the author is never the only possible approver.

<h4>8. "What is the difference between git fetch and git pull?"</h4>

<b>Model answer:</b> `fetch` downloads new commits and updates remote-tracking branches like `origin/main` without touching my working branch. `pull` is `fetch` plus a merge (or rebase with `--rebase`) into my current branch. I often fetch first, look at `git log HEAD..origin/main`, and then decide.

<h3>Live Quiz (10 questions)</h3>


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

<h3>Lab Briefing</h3>

<b>What you build this afternoon:</b> the shared batch repository `peoplepulse-team`, which will hold the team version of PeoplePulse for the rest of the program.

<b>Roles (example; the trainer may rotate them):</b>

| Person | Role today |
|---|---|
| Haiman | Team lead: creates the repo, adds collaborators, sets protection, tags the release |
| Cindia, Jobanjit | Developers: open feature PRs, cause and resolve the conflict |
| Carla, Raymundo | Reviewers (QA): review every PR, request changes at least once, own `/tests/` |

Everyone opens at least 2 PRs and reviews at least 2 PRs by the end of Lab Part 2.

<b>Acceptance criteria:</b>
- The repo has `README.md`, `.gitignore`, `.github/CODEOWNERS`, `.github/pull_request_template.md`, `data/employees.csv` and `CHANGELOG.md`.
- `main` is protected: PR required, 1 approval, code-owner review, stale approvals dismissed, conversation resolution, no force push or deletion, admins included. A direct `git push origin main` is <b>rejected</b>.
- At least 6 merged PRs with conventional-commit titles; at least one went through "changes requested" before approval.
- One merge conflict resolved inside a PR (by rebase).
- Annotated tag `v0.1.0` and a GitHub Release exist.
- Every trainee shows the mentor a reflog recovery.

<b>Time-boxes:</b> Tasks 1–3 by 1:45 PM, Task 4 by 2:05 PM, Tasks 5–6 by 2:30 PM, Lab Part 2 challenges 2:30–4:00 PM.

---


<h2>12:15 – 1:15 PM · Lunch</h2>
<img src="https://img.shields.io/badge/Break-12%3A15%20%E2%80%93%201%3A15%20PM-9E9E9E?style=flat-square" alt="Break: 12:15 – 1:15 PM">

Take a proper break away from the screen. Before 1:15 PM, accept the collaborator invitation email from GitHub as soon as it arrives.

---


<h2>1:15 – 2:45 PM · Lab Part 1 – Task Assignment & Guided Build</h2>
<img src="https://img.shields.io/badge/Lab-1%3A15%20%E2%80%93%202%3A45%20PM-2E7D32?style=flat-square" alt="Lab: 1:15 – 2:45 PM">


> [!NOTE]
> <b>Lab start (1:15 PM):</b> the trainer posts today's lab tasks in the batch channel. Work through them in order, ask for help in the channel if you are stuck for more than 15 minutes, and report your results (screenshots, command output, Git link) to the trainer at the <b>4:00 PM review</b>.


| Minutes | Activity | Method |
|---|---|---|
| 1:15 – 1:30 | Task 1: lead creates `peoplepulse-team`; everyone accepts the invite and clones | Task |
| 1:30 – 1:45 | Task 2: lead pushes the skeleton (the only direct push to `main`) | Task |
| 1:45 – 2:00 | Task 3: lead protects `main`; everyone proves a direct push is rejected | Task |
| 2:00 – 2:20 | Task 4: developers raise PRs; reviewers request changes, then approve | Task |
| 2:20 – 2:35 | Task 5: create and resolve a merge conflict inside a PR | Task |
| 2:35 – 2:43 | Task 6: tag and release `v0.1.0`; everyone does the reflog drill | Task |
| 2:43 – 2:45 | Mentor checks the protection settings and the PR timeline | Mentor check |

```mermaid
flowchart TB
  subgraph GH["GitHub repo peoplepulse-team"]
    Main["main<br/>protected"]
    CO[".github/CODEOWNERS<br/>PR template"]
    Data["data/employees.csv"]
    Rel["tag v0.1.0<br/>GitHub Release"]
  end
  Lead["Haiman<br/>lead"] -->|"skeleton, protection"| Main
  Devs["Cindia, Jobanjit<br/>developers"] -->|"PRs, conflict"| Main
  QA["Carla, Raymundo<br/>reviewers"] -->|"request changes, approve"| Main
  CO --> QA
  Main --> Rel
  Main --> Data
  class Lead,Devs,QA user
  class Main,Rel git
  class CO sec
  class Data data
  classDef aws fill:#FFE8CC,stroke:#FF9900,stroke-width:2px,color:#232F3E
  classDef azure fill:#DCEEFF,stroke:#0078D4,stroke-width:2px,color:#0B2A4A
  classDef k8s fill:#E3E9FB,stroke:#326CE5,stroke-width:2px,color:#13284F
  classDef git fill:#FDE5DD,stroke:#F05032,stroke-width:2px,color:#4A1408
  classDef cicd fill:#FFF3C4,stroke:#D4A017,stroke-width:2px,color:#4A3800
  classDef user fill:#E8F5E9,stroke:#2E7D32,stroke-width:2px,color:#1B3D1E
  classDef sec fill:#FDE7E7,stroke:#C62828,stroke-width:2px,color:#4A0E0E
  classDef mon fill:#F3E5F5,stroke:#8E24AA,stroke-width:2px,color:#3A0F45
  classDef data fill:#E0F2F1,stroke:#00897B,stroke-width:2px,color:#0B3B36
  classDef linux fill:#F1F1F1,stroke:#424242,stroke-width:2px,color:#212121
  classDef app fill:#E8EAF6,stroke:#3949AB,stroke-width:2px,color:#1A237E
```
<p align="center"><sub><i>Figure: Lab - what you will build this afternoon</i></sub></p>


Reading the diagram: Haiman sets up the repo and protects `main`. Cindia and Jobanjit change the code only through pull requests, and CODEOWNERS sends review requests to Carla and Raymundo, who must approve before anything merges. At the end, `main` is tagged and released as `v0.1.0`.

<h3>Task 1 – Lead: Create the Team Repo and Add Teammates</h3>

Haiman (lead) runs:

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

Everyone else accepts the invitation (email link, or `https://github.com/<gh-haiman>/peoplepulse-team/invitations`) and clones:

```bash
cd ~/devops-8wk
git clone git@github.com:<gh-haiman>/peoplepulse-team.git && cd peoplepulse-team
git log --oneline
```

```text
1f0e9d8 (HEAD -> main, origin/main, origin/HEAD) Initial commit
```

<b>Checkpoint:</b> all five can `git pull`; <b>Settings → Collaborators</b> shows four accepted collaborators.

<h3>Task 2 – Lead: Initial Skeleton (the Only Direct Push to main)</h3>


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

```text
[main 2a3b4c5] chore: repo skeleton, codeowners, pr template, seed data
 5 files changed, 48 insertions(+)
To github.com:<gh-haiman>/peoplepulse-team.git
   1f0e9d8..2a3b4c5  main -> main
```

<b>Checkpoint:</b> the files are visible on GitHub, and the CODEOWNERS page shows no errors (GitHub underlines invalid owners in the file view).

> [!WARNING]
> <b>Common mistake:</b> Push the skeleton <b>before</b> you protect `main`. After Task 3, even the lead cannot push directly. That is the whole point.


<h3>Task 3 – Lead: Protect main</h3>

<b>Option A – web UI:</b> <b>Settings → Branches → Add classic branch protection rule</b> (or <b>Settings → Rules → Rulesets → New branch ruleset</b>) for `main`, with every rule from concept 9.

<b>Option B – CLI</b> (classic protection through the REST API):

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

```json
{"admins":true,"codeowners":true,"reviews":1}
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

Now <b>everyone</b> proves it by trying a direct push:

```bash
git pull
echo "direct" >> README.md && git commit -qam "test: direct push" && git push origin main
```

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

<b>Checkpoint:</b> every trainee sees `GH006` and screenshots it for the report.

<h3>Task 4 – Developers: Raise PRs; Reviewers: Request Changes and Approve</h3>

<b>Cindia:</b> describe the project in the README:

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

<b>Jobanjit:</b> add a contributing guide:

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

<b>Carla and Raymundo:</b> review. Carla requests a change on the first PR; Cindia fixes it on the same branch; Carla approves:

```bash
gh pr list
gh pr diff 1
gh pr review 1 --request-changes --body "Please mention that the data lives in data/employees.csv and add the port (8000)."
```

Cindia pushes a fix to the same branch, and the PR updates automatically:

```bash
git switch docs/readme-overview
echo "The API listens on port 8000. Data file: data/employees.csv (set with DATA_FILE)." >> README.md
git commit -am "docs: add port and data file location"
git push
```

Carla approves and merges; Raymundo reviews and merges PR 2:

```bash
gh pr review 1 --approve --body "Thanks, LGTM"
gh pr merge 1 --squash --delete-branch
gh pr review 2 --approve --body "Clear and short - LGTM"
gh pr merge 2 --squash --delete-branch
git switch main && git pull && git log --oneline -3
```

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

<b>Checkpoint:</b> PR no. 1's timeline shows "changes requested" → new commit → "approved" → "merged". `git log --oneline origin/main` shows one squash commit per PR with a conventional title and the PR number.

<h3>Task 5 – Resolve a Merge Conflict in a PR</h3>

Cindia and Jobanjit both branch from the <b>same</b> `main` and add employee ID 4 at the same time:

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

Raymundo approves and merges Cindia's PR first. Jobanjit's PR now shows <b>"This branch has conflicts that must be resolved"</b>. Jobanjit rebases onto the new `main`:

```bash
git fetch origin
git rebase origin/main
```

```text
Auto-merging data/employees.csv
CONFLICT (content): Merge conflict in data/employees.csv
error: could not apply 7f3e2d1... feat(data): add Raymundo Trancoso
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
```

Edit `data/employees.csv`: keep Carla as id 4, change Raymundo to id 5, delete all three marker lines. Then:

```bash
git diff --check
git add data/employees.csv
git rebase --continue          # opens the editor for the message - save and close
git push --force-with-lease
tail -2 data/employees.csv
```

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

<b>Checkpoint:</b> Jobanjit's PR shows no conflicts, gets approved by Carla and merged; `data/employees.csv` on `main` has ids 1–5 and `grep -c '<<<<' data/employees.csv` prints `0`.

<h3>Task 6 – Tag v0.1.0 and Recover a Lost Commit</h3>

<b>Haiman (lead):</b> changelog via PR, then tag and release (tags are not blocked by branch protection):

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

```text
https://github.com/<gh-haiman>/peoplepulse-team/releases/tag/v0.1.0
v0.1.0 v0.1.0
```

<b>Everyone:</b> the reflog drill in your own clone:

```bash
git switch -c scratch/reflog-drill
echo "draft notes" > notes.txt && git add notes.txt && git commit -qm "docs: draft notes"
git switch main
git branch -D scratch/reflog-drill        # "oops" - branch and commit gone
git reflog | grep "draft notes"
git branch scratch/reflog-drill <hash>    # use the hash from the line above
git log --oneline -1 scratch/reflog-drill
```

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

<b>Checkpoint:</b> `gh release view v0.1.0` works; each trainee shows the mentor their recovered `docs: draft notes` commit.

---


<h2>2:45 – 4:00 PM · Lab Part 2 – Independent Challenge</h2>
<img src="https://img.shields.io/badge/Lab-2%3A45%20%E2%80%93%204%3A00%20PM-2E7D32?style=flat-square" alt="Lab: 2:45 – 4:00 PM">

| Minutes | Activity | Method |
|---|---|---|
| 2:45 – 3:06 | Challenge 1: squash a messy branch into one clean commit | Task |
| 3:06 – 3:27 | Challenge 2: hotfix on `main` and cherry-pick to `release/0.1` | Task |
| 3:27 – 3:43 | Challenge 3: break/fix – a secret was committed | Task |
| 3:43 – 3:56 | Challenge 4: release `v0.2.0` from conventional commits | Task |
| 3:56 – 4:00 | Mentor spot-checks Challenge 3 history; stretch goal if time | Mentor check |

These challenges have <b>no full solutions</b>. Work from the hints, the concept notes and `git log --graph --all`.

<h3>Challenge 1 – Clean Up a Messy Branch Before Review</h3>

<b>Goal:</b> on a new branch, make 4 commits named `wip`, `fix typo`, `more` and `final final` that together add `docs/branching.md` (describe our trunk-based flow in 5 lines). Before opening the PR, turn them into <b>one</b> conventional commit.

<b>Hints:</b> `git rebase -i origin/main` and mark the last three commits as `fixup` (or `squash`); or `git reset --soft origin/main` then commit once. If you already pushed, update with `--force-with-lease`.

<b>Acceptance criteria:</b> the PR shows exactly one commit, `docs: describe trunk-based branching`, and is merged after review.

<h3>Challenge 2 – Hotfix and Cherry-Pick</h3>

<b>Goal:</b> Carla's email in `employees.csv` must be `carla.rivera@peoplepulse.example.com` in `v0.1.0` and later. Fix it on `main` through a PR, release `v0.1.1`, and also apply the same fix to a long-lived `release/0.1` branch with `git cherry-pick`.

<b>Hints:</b> create the release branch from the tag: `git switch -c release/0.1 v0.1.0`, push it, and protect it too if you like. Find the fix commit's hash on `main` with `git log --oneline -3`, then `git cherry-pick <sha>` on `release/0.1`. Which semver number changes for a fix?

<b>Acceptance criteria:</b> tag `v0.1.1` exists; `git log release/0.1 --oneline` shows the fix; you can explain why this is a PATCH release.

<h3>Challenge 3 – Break/Fix: a Secret Was Committed</h3>

Your mentor (or you) commits a <b>fake</b> secret on a feature branch:

```bash
git switch main && git pull
git switch -c feat/db-config
echo 'DB_PASSWORD=Sup3rS3cret-fake-lab' > db.prod.env
git add db.prod.env && git commit -m "feat: add db config"
echo 'DATABASE_HOST=db.internal' > config.txt && git add config.txt && git commit -m "feat: add config"
```

<b>Goal:</b> the secret must not exist in <b>any</b> commit of the branch, while `config.txt` is kept. (In real life, step 1 is always: rotate the password.)

<b>Hints:</b>
- `git log -p --all -S 'DB_PASSWORD'` finds every commit that added or removed the string.
- For a branch that was never pushed, an interactive rebase (`git rebase -i origin/main`) that drops or edits the first commit is enough.
- For pushed history, use `git filter-repo --path db.prod.env --invert-paths` (install with `pip install git-filter-repo`) on a fresh clone.
- Add a pattern to `.gitignore` so `*.env` files cannot be added again by accident.

<b>Acceptance criteria:</b> `git log -p feat/db-config -S 'Sup3rS3cret'` returns nothing; `config.txt` is still on the branch; `.gitignore` covers `*.env`; your report has a one-paragraph incident note (detect, rotate, clean, prevent with secret scanning push protection).

> [!CAUTION]
> <b>Cost / security alert:</b> In a real incident, cleaning Git history does <b>not</b> make the secret safe: forks, clones, CI logs and GitHub's cache may still have it. Rotating the credential is the only real fix. Cleaning history only stops the problem from spreading further.


<h3>Challenge 4 – Release v0.2.0 from Conventional Commits</h3>

<b>Goal:</b> Cindia and Jobanjit each merge one `feat(...)` PR. For example, add a sixth column note to `docs/data-dictionary.md` and an `docs/architecture.md` describing the future API. Decide the next version from the commit types since `v0.1.1`, update `CHANGELOG.md` through a PR, then tag and release.

<b>Hints:</b> `git log v0.1.1..origin/main --oneline` lists what changed since the last release.

<b>Acceptance criteria:</b> a `v0.2.0` release with generated notes; the changelog lists the features; you can explain why it is MINOR and not PATCH.

<h3>Stretch Goal</h3>

Replace the classic protection with a <b>repository ruleset</b> that protects `main` <b>and</b> blocks deletion and updates of tags matching `v*`. Then turn on <b>secret scanning push protection</b> (<b>Settings → Code security</b>) and try to push a file containing a fake AWS key pattern from the AWS documentation (`AKIAIOSFODNN7EXAMPLE`) on a scratch branch. Record what GitHub does in your notes.

---


<h2>4:00 – 4:45 PM · Lab Review with Trainer & Teardown</h2>
<img src="https://img.shields.io/badge/Lab-4%3A00%20%E2%80%93%204%3A45%20PM-2E7D32?style=flat-square" alt="Lab: 4:00 – 4:45 PM">

| Minutes | Activity | Method |
|---|---|---|
| 4:00 – 4:25 | Mentor reviews the repo settings, PR history and each trainee's reflog drill | Mentor check |
| 4:25 – 4:38 | Teardown: delete merged branches and the demo repos | Task |
| 4:38 – 4:45 | Cost check | Checkpoint |

<h3>Mentor Review Checklist</h3>

- `main` protection: PR required, 1 approval, code owners, dismiss stale reviews, conversation resolution, admins included, no force push or deletion.
- `GH006` direct-push rejection demonstrated by every trainee.
- At least 6 merged PRs with conventional titles; at least one "changes requested" cycle.
- Conflict resolved correctly: no markers, unique IDs, readable history (`git log --graph`).
- `v0.1.0` annotated tag and release (plus `v0.1.1` and `v0.2.0` if done).
- Every trainee did a reflog recovery.
- Secret challenge: string not found in branch history; `.gitignore` updated; incident note written.
- `.gitignore` covers `.venv`, `.env`, `*.pem`, `*.tfstate`.

<h3>Teardown Commands</h3>

Keep the team repo, because the team works in it again in later weeks. Clean up branches and the local demo folders:

```bash
cd ~/devops-8wk/peoplepulse-team
git switch main && git pull --prune
git branch --merged main | grep -vE '^\*|main|release/' | xargs -r git branch -d
git push origin --delete feat/db-config 2>/dev/null || true
gh pr list --state open
rm -rf ~/day04-demo ~/day04-areas
```

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

<h3>Cost Check</h3>

GitHub Free covers everything today (public repo, unlimited collaborators). You created no cloud resources. If you started a paid trial (GitHub Team or Advanced Security) for the stretch goal, write the trial end date in your report and cancel it if you do not need it.

---


<h2>4:45 – 5:00 PM · Commit & Daily Report</h2>
<img src="https://img.shields.io/badge/Lab-4%3A45%20%E2%80%93%205%3A00%20PM-2E7D32?style=flat-square" alt="Lab: 4:45 – 5:00 PM">

| Minutes | Activity | Method |
|---|---|---|
| 4:45 – 4:53 | Write lab notes in your <b>personal</b> repo on branch `day04-git-github`, push, open a PR | Task |
| 4:53 – 5:00 | Fill in and post the daily activity report | Checkpoint |

The team repo already holds today's PR history. In your personal training repo, record what you did:

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

Merge the PR after the mentor review.

<b>Daily activity report – Day 04:</b>

```text
Done
- Built peoplepulse-team with CODEOWNERS, PR template, .gitignore and seed employees.csv
- Protected main (PR + 1 approval + code owners, no force push, admins included); direct push rejected (GH006)
- Opened <n> PRs and reviewed <n> (one with changes requested); resolved the employees.csv conflict by rebase
- Tagged and released v0.1.0; recovered a deleted branch with git reflog

Blocked
- <e.g. protection not enforced on a private repo - switched to public / none>

Next
- Day 5: Python for DevOps - build the PeoplePulse CSV Records API with FastAPI + pytest; weekly assessment

Learned
- Branches are just pointers; reflog can recover "lost" commits
- Rebase private branches, merge shared ones; use --force-with-lease
- feat -> MINOR, fix -> PATCH, BREAKING CHANGE -> MAJOR
```

---


<h2>Cheat Sheet</h2>

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


<h2>Further Reading</h2>

- [Pro Git book – Git Internals (git-scm.com)](https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain)
- [GitHub Docs – About protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)
- [GitHub Docs – About rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets)
- [GitHub Docs – About code owners](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners)
- [Semantic Versioning 2.0.0](https://semver.org/)
- [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/)

<b>Next:</b> [Day 05 – Python for DevOps + Mini Project 1 start](../Day05_Python_FastAPI_and_MiniProject1_Start/Day05_Python_FastAPI_and_MiniProject1_Start.md)


<h2>Answer Key</h2>
<img src="https://img.shields.io/badge/Answer%20key-BF9000?style=flat-square" alt=": Answer key">

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
