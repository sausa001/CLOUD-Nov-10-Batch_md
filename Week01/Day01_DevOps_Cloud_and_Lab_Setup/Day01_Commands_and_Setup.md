<h1 align="left">Day 01 – DevOps, Cloud & Lab Setup – Commands & Setup Steps</h1>

<p>
<img src="https://img.shields.io/badge/Week-01-1F3864?style=flat-square" alt="Week: 01">
<img src="https://img.shields.io/badge/Day-01-2E75B6?style=flat-square" alt="Day: 01">
<img src="https://img.shields.io/badge/Cloud-AWS%20%2B%20Azure-FF9900?style=flat-square" alt="Cloud: AWS + Azure">
<img src="https://img.shields.io/badge/Class-8%3A00%E2%80%9312%3A15%20EST-2E75B6?style=flat-square" alt="Class: 8:00–12:15 EST">
<img src="https://img.shields.io/badge/Lab-1%3A15%E2%80%935%3A00%20EST-2E7D32?style=flat-square" alt="Lab: 1:15–5:00 EST">
</p>

<p><a href="Day01_DevOps_Cloud_and_Lab_Setup.md">Notes</a> · <b>Commands & Setup</b> · <a href="Day01_Questions_and_Answers.md">Questions & Answers</a> · <a href="../README.md">Week overview</a> · <a href="../../README.md">Course index</a></p>



Companion to the main notes: [Day01_DevOps_Cloud_and_Lab_Setup.md](Day01_DevOps_Cloud_and_Lab_Setup.md). Every command used today, in the order you meet it in class and lab,
with the command breakdown tables that explain each part. Run commands in your own terminal and replace
`<your-name>` with your resource name from the Batch Roster.


<h2>1. Installation & Setup Steps</h2>


Run these before or at the start of the lab (installs, logins, cluster start-up, tool configuration).

<b>From:</b> 1:15 – 2:45 PM · Lab Part 1 – Task Assignment & Guided Build → Task 1 – Prepare Your Operating System

```powershell
# Run PowerShell as Administrator
wsl --install -d Ubuntu-24.04
# Reboot, open "Ubuntu" from the Start menu, create a Linux username and password
wsl --list --verbose     # the VERSION column must show 2
```

<b>From:</b> 1:15 – 2:45 PM · Lab Part 1 – Task Assignment & Guided Build → Task 1 – Prepare Your Operating System

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install -y curl wget unzip gnupg ca-certificates lsb-release
```

<b>From:</b> 1:15 – 2:45 PM · Lab Part 1 – Task Assignment & Guided Build → Task 2 – Install the Toolchain

```bash
brew install git python@3.12 kubectl awscli azure-cli
brew tap hashicorp/tap && brew install hashicorp/tap/terraform
brew install --cask visual-studio-code docker
open -a Docker   # start Docker Desktop once and accept the terms
```

<b>From:</b> 1:15 – 2:45 PM · Lab Part 1 – Task Assignment & Guided Build → Task 2 – Install the Toolchain

```powershell
winget install -e --id Microsoft.VisualStudioCode
winget install -e --id Git.Git
winget install -e --id Docker.DockerDesktop
winget install -e --id Python.Python.3.12
winget install -e --id Kubernetes.kubectl
winget install -e --id Hashicorp.Terraform
winget install -e --id Amazon.AWSCLI
winget install -e --id Microsoft.AzureCLI
```

<b>From:</b> 1:15 – 2:45 PM · Lab Part 1 – Task Assignment & Guided Build → Task 2 – Install the Toolchain

```bash
# Git + Python 3.12 (Ubuntu 24.04 ships Python 3.12)
sudo apt-get install -y git python3 python3-venv python3-pip
python3 --version

# kubectl (pinned minor version v1.34 from the official pkgs.k8s.io repo)
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key \
  | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' \
  | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update && sudo apt-get install -y kubectl

# Terraform (HashiCorp apt repo)
wget -O- https://apt.releases.hashicorp.com/gpg \
  | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" \
  | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt-get update && sudo apt-get install -y terraform

# AWS CLI v2 (official installer; use the aarch64 zip on ARM machines)
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
unzip -q awscliv2.zip && sudo ./aws/install && rm -rf aws awscliv2.zip

# Azure CLI (official script for Debian/Ubuntu)
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

<b>From:</b> 1:15 – 2:45 PM · Lab Part 1 – Task Assignment & Guided Build → Task 2 – Install the Toolchain

```bash
git config --global user.name  "<Your Name>"
git config --global user.email "<your-github-email>"
git config --global init.defaultBranch main
git config --global --list        # check the values
```

<b>From:</b> 1:15 – 2:45 PM · Lab Part 1 – Task Assignment & Guided Build → Task 4 – Azure Account, MFA and Budget

```bash
az login                     # opens the browser; on WSL use: az login --use-device-code
az account show --query "{name:name, id:id, state:state}" --output table
```


<h2>2. Commands by Session</h2>


<h3>8:15 – 9:45 AM · Concept Session 1</h3>


<h4>1. What DevOps Is – Culture and CALMS</h4>


```bash
#!/usr/bin/env bash
# health-check.sh – a manual step turned into automation (the "A" in CALMS)
URL="${1:-https://example.com}"                  # URL to check, default example.com
CODE=$(curl -s -o /dev/null -w "%{http_code}" "$URL")   # get only the HTTP status code
if [ "$CODE" = "200" ]; then
  echo "OK   $URL returned $CODE"
else
  echo "FAIL $URL returned $CODE"
  exit 1                                          # non-zero exit fails a pipeline step
fi
```

<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `curl` | a tool that sends web (HTTP) requests from the terminal |
| `-s` | silent – hide the progress bar |
| `-o /dev/null` | throw the page body away; we only need the status code |
| `-w "%{http_code}"` | write out only the HTTP status code (200 OK, 404 not found, 500 server error) |
| `CODE=$(...)` | run the command inside `$( )` and save its output in the variable `CODE` |
| `${1:-https://example.com}` | use the first argument; if none is given, use the default URL |
| `exit 1` | end with a failure code; a pipeline step that exits non-zero turns red |

<h4>2. The SDLC and How DevOps Changes It</h4>


```bash
git log --oneline -5      # show the last 5 commits, one line each
```

Expected output:

```text
a1c9e3f fix: return 404 for unknown record id
7b2d410 test: add test for /records/{id}
c4e8a21 feat: add /records/{id} endpoint
19f0b7d chore: pin fastapi to 0.115.0
e02a6c3 ci: run pytest on every pull request
```

<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `git log` | show the commit history of the current branch |
| `--oneline` | one line per commit: short ID + message |
| `-5` | only the last 5 commits |

<h4>3. CI vs Continuous Delivery vs Continuous Deployment</h4>


<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `pytest` | the Python test runner; finds and runs files named `test_*.py` |
| `--cov=app` | measure how much of the `app` package the tests execute (coverage) |
| `--cov-fail-under=80` | fail the step if coverage is below 80% – this is the quality gate |
| `on: [push, pull_request]` | run the workflow on every push and every pull request |
| `uses: actions/checkout@v4` | a ready-made step pinned to version 4 that downloads the repo code |

<h4>4. Infrastructure as Code (IaC)</h4>


<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `terraform init` | run once first: downloads the AWS provider plugin |
| `terraform plan` | compare code with real infrastructure and show what <i>would</i> change – changes nothing |
| `+` | in the plan output means "will be created"; `~` means changed, `-` means destroyed |
| `terraform apply` | show the plan again and, after you type `yes`, make the changes |

<h4>5. GitOps</h4>


<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `kubectl get applications` | list Argo CD Application objects (a custom Kubernetes resource) |
| `-n argocd` | look in the `argocd` namespace, where Argo CD is installed |
| `Synced` | the cluster matches Git; `OutOfSync` means they differ |
| `Healthy` | the app's pods and services are running correctly |

<h4>7. DORA Metrics – With a Worked Calculation</h4>


<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `csv.DictReader(f)` | read each CSV row as a dictionary, e.g. `r["deploy_time"]` |
| `datetime.strptime(..., FMT)` | turn text like `2026-11-02 09:00` into a real date-time |
| `.total_seconds() / 3600` | convert the time difference to hours |
| `median(...)` | the middle value after sorting – not distorted by one slow deploy |
| `:.1%` | format a fraction like 0.333 as `33.3%` |

<h3>10:00 – 11:15 AM · Concept Session 2 + Live Demo</h3>


<h4>8. Cloud Computing and the Service Models (IaaS, PaaS, SaaS)</h4>


```bash
# AWS: details of the t3.micro instance type
aws ec2 describe-instance-types --instance-types t3.micro \
  --query "InstanceTypes[].{Type:InstanceType,vCPU:VCpuInfo.DefaultVCpus,MemoryMiB:MemoryInfo.SizeInMiB}" \
  --output table

# Azure: details of the Standard_B1s VM size in East US
az vm list-sizes --location eastus --query "[?name=='Standard_B1s']" --output table
```

<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `aws ec2 describe-instance-types` | ask EC2 for the details of instance (VM) types; read-only and free |
| `--instance-types t3.micro` | only the `t3.micro` type |
| `--query "InstanceTypes[].{...}"` | JMESPath filter: pick only the fields we want and rename them (Type, vCPU, MemoryMiB) |
| `--output table` | print a readable table instead of JSON |
| `az vm list-sizes --location eastus` | list the VM sizes available in the East US region |
| `--query "[?name=='Standard_B1s']"` | keep only the row whose name is `Standard_B1s` |

<h4>9. Regions and Availability Zones</h4>


```bash
aws ec2 describe-availability-zones --region us-east-1 \
  --query "AvailabilityZones[].ZoneName" --output text

az account list-locations \
  --query "[?name=='eastus'].{Name:name, Display:displayName, Paired:metadata.pairedRegion[0].name}" \
  --output table
```

<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `aws ec2 describe-availability-zones` | list the Availability Zones you can use |
| `--region us-east-1` | ask about N. Virginia, whatever your default region is |
| `--query "AvailabilityZones[].ZoneName"` | return only the zone names |
| `--output text` | print plain text separated by tabs – easy to use in scripts |
| `az account list-locations` | list all Azure regions |
| `metadata.pairedRegion[0].name` | the first paired region, used for disaster recovery |

<h4>10. The Shared Responsibility Model</h4>


```bash
# 1 = root user has MFA, 0 = root user has NO MFA (fix this today)
aws iam get-account-summary --query "SummaryMap.AccountMFAEnabled"
```

<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `aws iam get-account-summary` | return counts and flags for the whole account (users, roles, MFA…) |
| `--query "SummaryMap.AccountMFAEnabled"` | pick only the root-MFA flag |
| `1` / `0` | 1 = root MFA on; 0 = root MFA off – a security finding to fix today |

<h4>11. Pay-As-You-Go Pricing and Why Budgets Matter</h4>


```bash
# Month-to-date cost from AWS Cost Explorer (enable Cost Explorer once in the console first)
aws ce get-cost-and-usage \
  --time-period Start=2026-11-01,End=2026-11-09 \
  --granularity MONTHLY --metrics UnblendedCost \
  --query "ResultsByTime[].Total.UnblendedCost" --output table
```

<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `aws ce get-cost-and-usage` | query AWS Cost Explorer for cost data (each call costs about $0.01) |
| `--time-period Start=...,End=...` | date range; `End` is exclusive (not included) |
| `--granularity MONTHLY` | one total per month (other options: `DAILY`, `HOURLY`) |
| `--metrics UnblendedCost` | the actual cost you are charged, before discounts are spread |
| `--query "ResultsByTime[].Total.UnblendedCost"` | keep only the amount and currency |

<h4>12. AWS vs Azure at a Glance</h4>


```bash
aws sts get-caller-identity      # AWS: which account and identity am I using?
az account show --query "{subscription:name, user:user.name, state:state}" --output table
```

Expected output:

```text
{
    "UserId": "AIDAEXAMPLE123456789",
    "Account": "<ACCOUNT_ID>",
    "Arn": "arn:aws:iam::<ACCOUNT_ID>:user/<your-name>-admin"
}
Subscription          User             State
--------------------  ---------------  -------
Azure subscription 1  you@example.com  Enabled
```

<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `aws sts get-caller-identity` | ask AWS STS (Security Token Service) which identity your credentials belong to; works even with zero permissions |
| `az account show` | show the Azure subscription the CLI is currently using |
| `--query "{subscription:name, user:user.name, state:state}"` | build a small object with three renamed fields |
| `--output table` | print it as a table |

<h4>14. A Day in the Life of a DevOps Engineer</h4>


```bash
kubectl get pods -n peoplepulse                      # are all app pods running?
aws cloudwatch describe-alarms --state-value ALARM \
  --query "MetricAlarms[].AlarmName" --output text   # any alarms firing right now?
```

Expected output:

```text
NAME                           READY   STATUS    RESTARTS   AGE
peoplepulse-7c9d8f6b5d-4xk2p   1/1     Running   0          2d
peoplepulse-7c9d8f6b5d-9hq7m   1/1     Running   0          2d
```

<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `kubectl get pods` | list pods (running containers) in Kubernetes |
| `-n peoplepulse` | in the `peoplepulse` namespace |
| `READY 1/1` | 1 of 1 containers in the pod is ready |
| `aws cloudwatch describe-alarms` | list CloudWatch alarms |
| `--state-value ALARM` | only alarms that are firing right now (not `OK` or `INSUFFICIENT_DATA`) |

<h4>Live Demo: Verify the Toolchain, Check Identities and Create an AWS Budget from the CLI</h4>


```bash
git --version
python3 --version              # macOS with Homebrew: python3.12 --version
docker --version
docker compose version
kubectl version --client
terraform version
aws --version
az version --output table
code --version
```

Expected output:

```text
git version 2.47.1
Python 3.12.7
Docker version 28.5.1, build e180ab8
Docker Compose version v2.40.0
Client Version: v1.34.1
Kustomize Version: v5.7.1
Terraform v1.13.4
on linux_amd64
aws-cli/2.31.20 Python/3.13.7 Linux/6.8.0 exe/x86_64.ubuntu.24
Azure-cli    Azure-cli-core    Azure-cli-telemetry
-----------  ----------------  ---------------------
2.78.0       2.78.0            1.1.0
1.105.1
e0e9ec3a2d0bf3e8f1d8c2f0b4a9a6c5d3e7f812
x64
```

<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `--version` | most tools print their version with this flag |
| `docker compose version` | Compose v2 is a Docker plugin, so it is two words with a space, not `docker-compose` |
| `kubectl version --client` | show only the local kubectl version; without `--client` it also tries to reach a cluster and prints an error |
| `az version --output table` | Azure CLI versions as a table |
| `code --version` | VS Code version, commit ID and CPU architecture (3 lines) |

```bash
docker run --rm hello-world
```

Expected output:

```text
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `docker run` | create and start a container from an image |
| `--rm` | delete the container automatically when it exits, so stopped containers do not pile up |
| `hello-world` | a tiny official test image; Docker downloads it from Docker Hub the first time |

```bash
aws configure sso          # if your account uses IAM Identity Center (recommended)
# or: aws configure        # access key of an IAM user (lab only, never root keys)
aws sts get-caller-identity
```

<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `aws configure sso` | interactive setup for IAM Identity Center: asks for your start URL and region, opens a browser to log in, and saves a named profile |
| `aws configure` | saves an access key ID, secret key, region and output format in `~/.aws/credentials` and `~/.aws/config` |
| `aws sts get-caller-identity` | prints the account and identity in use; if the ARN ends in `:root`, stop and switch identity |

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

cat > budget.json <<'EOF'
{
  "BudgetName": "devops-8wk-monthly",
  "BudgetLimit": { "Amount": "25", "Unit": "USD" },
  "TimeUnit": "MONTHLY",
  "BudgetType": "COST"
}
EOF

cat > notifications.json <<'EOF'
[
  {
    "Notification": { "NotificationType": "ACTUAL", "ComparisonOperator": "GREATER_THAN",
                      "Threshold": 40, "ThresholdType": "PERCENTAGE" },
    "Subscribers": [ { "SubscriptionType": "EMAIL", "Address": "you@example.com" } ]
  },
  {
    "Notification": { "NotificationType": "ACTUAL", "ComparisonOperator": "GREATER_THAN",
                      "Threshold": 100, "ThresholdType": "PERCENTAGE" },
    "Subscribers": [ { "SubscriptionType": "EMAIL", "Address": "you@example.com" } ]
  },
  {
    "Notification": { "NotificationType": "FORECASTED", "ComparisonOperator": "GREATER_THAN",
                      "Threshold": 100, "ThresholdType": "PERCENTAGE" },
    "Subscribers": [ { "SubscriptionType": "EMAIL", "Address": "you@example.com" } ]
  }
]
EOF

aws budgets create-budget \
  --account-id "$ACCOUNT_ID" \
  --budget file://budget.json \
  --notifications-with-subscribers file://notifications.json
```

<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `ACCOUNT_ID=$(...)` | run the command and save the 12-digit account ID in a shell variable |
| `--query Account --output text` | return only the account number as plain text, without quotes |
| `cat > budget.json <<'EOF'` | write the following lines into `budget.json` until the line `EOF`; the quotes stop the shell from changing `$` signs |
| `"TimeUnit": "MONTHLY"` | the budget resets every month |
| `"BudgetType": "COST"` | track money spent (other types track usage or savings plans) |

<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `"NotificationType": "ACTUAL"` | alert on money already spent |
| `"NotificationType": "FORECASTED"` | alert when AWS predicts you <i>will</i> go over by month end |
| `"Threshold": 40` + `PERCENTAGE` | 40% of $25 = $10 |
| `"SubscriptionType": "EMAIL"` | send the alert by email (SNS is the other option) |

<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `aws budgets create-budget` | create the budget and its alerts in one call |
| `--account-id "$ACCOUNT_ID"` | which account the budget belongs to (required) |
| `--budget file://budget.json` | read the budget settings from the local file; forgetting `file://` gives a JSON parse error |
| `--notifications-with-subscribers file://notifications.json` | attach the three alerts and their email subscribers |

```bash
aws budgets describe-budgets --account-id "$ACCOUNT_ID" \
  --query "Budgets[].{Name:BudgetName,Limit:BudgetLimit.Amount,Unit:BudgetLimit.Unit}" \
  --output table
```

Expected output:

```text
-------------------------------------------
|             DescribeBudgets             |
+--------+----------------------+---------+
|  Limit |        Name          |  Unit   |
+--------+----------------------+---------+
|  25.0  |  devops-8wk-monthly  |  USD    |
+--------+----------------------+---------+
```

<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `aws budgets describe-budgets` | list all budgets in the account |
| `--query "Budgets[].{Name:...,Limit:...,Unit:...}"` | for every budget, show only its name, limit amount and currency |
| `BudgetLimit.Amount` | a nested field: the `Amount` inside `BudgetLimit` |

```bash
az login                        # opens a browser; on WSL use: az login --use-device-code
az account show --query "{name:name, state:state}" --output table
az consumption budget list --query "[].{name:name, amount:amount, grain:timeGrain}" --output table
```

Expected output:

```text
Name                  State
--------------------  -------
Azure subscription 1  Enabled
Name                Amount    Grain
------------------  --------  -------
devops-8wk-monthly  25.0      Monthly
```

<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `az login` | sign in to Azure; opens your browser and stores a token locally |
| `--use-device-code` | prints a code to type at microsoft.com/devicelogin – use it on WSL or servers without a browser |
| `az consumption budget list` | list budgets on the current subscription |
| `timeGrain` | how often the budget resets: `Monthly`, `Quarterly` or `Annually` |

<h3>1:15 – 2:45 PM · Lab Part 1 – Task Assignment & Guided Build</h3>


<h4>Task 1 – Prepare Your Operating System</h4>


<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `wsl --install` | turn on WSL and the Virtual Machine Platform, and install a Linux distribution |
| `-d Ubuntu-24.04` | which distribution to install (`wsl --list --online` shows all names) |
| `wsl --list --verbose` | list installed distributions with their state and WSL version; VERSION 1 is old and Docker will not work with it |

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew --version
```

<h4>Task 2 – Install the Toolchain</h4>


<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `brew install` | install a command-line package (a "formula") |
| `python@3.12` | a specific Python version; the command is then `python3.12` |
| `brew tap hashicorp/tap` | add HashiCorp's own package list, where the latest Terraform lives |
| `--cask` | install a desktop app (VS Code, Docker Desktop) instead of a CLI tool |
| `open -a Docker` | start the Docker Desktop app; the `docker` command fails until it is running |

<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `winget install` | Windows Package Manager: download and install an app |
| `-e` | exact match – install only the package with this exact ID |
| `--id Hashicorp.Terraform` | the package ID; find IDs with `winget search terraform` |

<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `curl -fsSL <url>` | download quietly (`-s`), fail on HTTP errors (`-f`), show errors (`-S`), follow redirects (`-L`) |
| pipe `\|` | send the output of the left command into the right command |
| `sudo gpg --dearmor -o <file>` | convert the signing key to binary format and save it where apt can find it |
| `[signed-by=...]` | apt only trusts packages from this repo if they are signed with that key |
| `sudo tee <file>` | write the line into a root-owned file (`>` alone would fail because of permissions) |
| `$(lsb_release -cs)` | your Ubuntu codename, e.g. `noble` for 24.04 |
| `unzip -q` | unzip quietly; `sudo ./aws/install` copies the AWS CLI to `/usr/local/bin` |
| `curl -sL ... \| sudo bash` | download Microsoft's install script and run it as root – only do this with official, trusted URLs |

```bash
sudo usermod -aG docker "$USER"   # run docker without sudo; log out and back in after this
```

<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `usermod` | modify a Linux user account |
| `-aG docker` | append (`-a`) the user to the group (`-G`) `docker`; without `-a` you are <i>removed</i> from all other groups |
| `"$USER"` | your own username |

<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `git config` | read or write Git settings |
| `--global` | save in `~/.gitconfig` for all your repos (without it, only the current repo) |
| `user.name` / `user.email` | the author shown on every commit; use the email linked to your GitHub account |
| `init.defaultBranch main` | new repos start on `main` instead of `master` |
| `--list` | print all settings so you can check them |

<h4>Task 3 – AWS Account, MFA, Admin Identity and Budget</h4>


```bash
aws sts get-caller-identity --query Arn --output text
aws budgets describe-budgets --account-id "$(aws sts get-caller-identity --query Account --output text)" \
  --query "Budgets[].BudgetName" --output text
aws iam get-account-summary --query "SummaryMap.AccountMFAEnabled"
```

Expected output:

```text
arn:aws:iam::<ACCOUNT_ID>:user/<your-name>-admin
devops-8wk-monthly
1
```

<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `aws configure set region us-east-1` | save `us-east-1` as the default region in `~/.aws/config` |
| `aws budgets describe-budgets --account-id "$(...)"` | the inner command gets your account ID and passes it straight in |
| `--query "Budgets[].BudgetName"` | only the budget names |

<h4>Task 4 – Azure Account, MFA and Budget</h4>


```bash
az consumption budget list --query "[].{name:name, amount:amount, grain:timeGrain}" --output table
```

Expected output:

```text
Name                Amount    Grain
------------------  --------  -------
devops-8wk-monthly  25.0      Monthly
```

<h3>4:00 – 4:45 PM · Lab Review with Trainer & Teardown</h3>


<h4>Teardown Commands</h4>


```bash
aws iam list-access-keys --user-name course-readonly-test --query "AccessKeyMetadata[].AccessKeyId" --output text
aws iam delete-access-key --user-name course-readonly-test --access-key-id <ACCESS_KEY_ID>
aws iam remove-user-from-group --user-name course-readonly-test --group-name course-readonly
aws iam delete-user --user-name course-readonly-test
aws configure list-profiles      # if "readonly" is listed, delete its block from ~/.aws/credentials and ~/.aws/config
```

<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `aws iam list-access-keys --user-name ...` | show the access key IDs of that user (you need the ID to delete it) |
| `aws iam delete-access-key` | delete one key; a user cannot be deleted while it still has keys |
| `aws iam remove-user-from-group` | take the user out of the group; also required before deletion |
| `aws iam delete-user` | delete the user; fails with `DeleteConflict` if keys or groups remain |
| `aws configure list-profiles` | list the named profiles saved on your laptop |

```bash
aws ec2 describe-instances --region us-east-1 \
  --query "Reservations[].Instances[].InstanceId" --output text     # expect empty
az resource list --output table                                     # expect empty
```

<h3>4:45 – 5:00 PM · Commit & Daily Report</h3>


```bash
mkdir -p ~/devops-8wk && cd ~/devops-8wk
git clone https://github.com/<your-github-user>/devops-8wk-<your-name>.git
cd devops-8wk-<your-name>
git switch -c day01-devops-setup
mkdir -p day01/screenshots
cp ~/tool-checklist.md ~/check-tools.sh ~/dora.md day01/ 2>/dev/null
# copy your 2 budget screenshots into day01/screenshots/ (check they show no account IDs or keys)
git add day01
git commit -m "docs(day01): tool checklist, budget alerts and DORA exercise"
git push -u origin day01-devops-setup
```

<b>Command breakdown:</b>

| Part | What it means |
|---|---|
| `git clone <url>` | download the repo from GitHub to your laptop |
| `git switch -c day01-devops-setup` | create (`-c`) a new branch and move to it, following the `dayNN-<topic>` convention |
| `git add day01` | stage the `day01` folder for the next commit |
| `git commit -m "..."` | save a snapshot with a message (`docs(day01): ...` follows the Conventional Commits style) |
| `git push -u origin day01-devops-setup` | upload the branch; `-u` remembers the link so later you can just type `git push` |
| `2>/dev/null` | hide error messages, e.g. if one of the files does not exist yet |


<h2>3. Cheat Sheet</h2>


| Purpose | Command |
|---|---|
| Check versions | `git --version`, `python3 --version`, `docker --version`, `kubectl version --client`, `terraform version`, `aws --version`, `az version`, `code --version` |
| Test Docker | `docker run --rm hello-world` |
| Who am I on AWS? | `aws sts get-caller-identity` |
| AWS SSO login | `aws configure sso`, then `aws sso login --profile <profile>` |
| Set AWS default region | `aws configure set region us-east-1` |
| Root MFA check | `aws iam get-account-summary --query "SummaryMap.AccountMFAEnabled"` |
| List AWS budgets | `aws budgets describe-budgets --account-id <ACCOUNT_ID>` |
| List AWS AZs | `aws ec2 describe-availability-zones --region us-east-1` |
| Azure login | `az login` (`--use-device-code` on WSL) |
| Current Azure subscription | `az account show -o table` |
| List Azure budgets | `az consumption budget list -o table` |
| List Azure resources | `az resource list -o table` |
| Git identity | `git config --global user.name "<Your Name>"` |

```text
DORA:   Deployment frequency | Lead time for changes | Change failure rate | Time to restore
CALMS:  Culture | Automation | Lean | Measurement | Sharing
Error budget (minutes per 30 days) = 43200 x (1 - SLO)     e.g. 99.9% -> 43.2 min
Monthly cost of a resource ~ hourly price x 730
IaaS: you manage OS and up | PaaS: you manage code | SaaS: you manage users and data
```
