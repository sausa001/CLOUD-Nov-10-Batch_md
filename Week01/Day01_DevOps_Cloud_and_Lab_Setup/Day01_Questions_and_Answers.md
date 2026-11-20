<h1 align="left">Day 01 – DevOps, Cloud & Lab Setup – Questions & Answers</h1>

<p>
<img src="https://img.shields.io/badge/Week-01-1F3864?style=flat-square" alt="Week: 01">
<img src="https://img.shields.io/badge/Day-01-2E75B6?style=flat-square" alt="Day: 01">
<img src="https://img.shields.io/badge/Cloud-AWS%20%2B%20Azure-FF9900?style=flat-square" alt="Cloud: AWS + Azure">
<img src="https://img.shields.io/badge/Class-8%3A00%E2%80%9312%3A15%20EST-2E75B6?style=flat-square" alt="Class: 8:00–12:15 EST">
<img src="https://img.shields.io/badge/Lab-1%3A15%E2%80%935%3A00%20EST-2E7D32?style=flat-square" alt="Lab: 1:15–5:00 EST">
</p>

<p><a href="Day01_DevOps_Cloud_and_Lab_Setup.md">Notes</a> · <a href="Day01_Commands_and_Setup.md">Commands & Setup</a> · <b>Questions & Answers</b> · <a href="../README.md">Week overview</a> · <a href="../../README.md">Course index</a></p>



Companion to the main notes: [Day01_DevOps_Cloud_and_Lab_Setup.md](Day01_DevOps_Cloud_and_Lab_Setup.md). All questions from today in one place: recap, interview Q&A,
live quiz and the answer key. Use it to revise and to prepare for client interviews.



<h2>2. Interview-style Questions with Model Answers</h2>


<h3>Q1. "What is DevOps, in your own words?"</h3>

DevOps is a culture and set of practices that lets one team deliver software quickly <i>and</i> safely. We work in small batches, automate build, test, infrastructure and deployment, and measure results with metrics like DORA. On my last project that meant a CI/CD pipeline with automated tests and Terraform-managed environments, so we could deploy daily instead of monthly.

<h3>Q2. "What is the difference between Continuous Delivery and Continuous Deployment?"</h3>

Both build and test every change automatically. With Continuous Delivery every passing change is releasable, but a person approves the production release. With Continuous Deployment it goes to production automatically. Regulated clients usually choose Delivery for the audit trail.

<h3>Q3. "Which metrics would you use to show your DevOps work is improving things?"</h3>

The four DORA metrics: deployment frequency and lead time for changes for speed; change failure rate and time to restore for stability. I baseline them first, then report the trend every sprint.

<h3>Q4. "How is SRE different from DevOps?"</h3>

DevOps is the broad culture. SRE is a concrete way to implement it with a focus on reliability: define SLIs and SLOs, use error budgets to balance features against stability, run blameless post-mortems and keep toil low with automation.

<h3>Q5. "What is GitOps and why would you use it?"</h3>

Git is the single source of truth for the desired state, and an agent like Argo CD pulls and reconciles the cluster to match. Benefits: every change is a reviewed PR, rollback is a `git revert`, drift is corrected automatically, and the CI system does not need production credentials.

<h3>Q6. "Explain the shared responsibility model."</h3>

The cloud provider secures the cloud itself: data centers, hardware and the hypervisor. The customer secures what they put in the cloud: identities and MFA, data, network rules and, for IaaS, OS patching. The more managed the service, the more the provider handles, but identity and data are always the customer's job.

<h3>Q7. "You get access to a new AWS account. What do you do first?"</h3>

Enable MFA on root and lock the root user away, create a non-root admin through IAM Identity Center, set up budgets and billing alerts, confirm CloudTrail is on, and check I am in the right account and region with `aws sts get-caller-identity` and `aws configure get region`.

<h3>Q8. "Why would you put an application in more than one Availability Zone?"</h3>

Each AZ is a separate data center with its own power and network. Running copies in two or three AZs behind a load balancer means one data center failure does not cause an outage. It costs a little more, so we match it to the SLO.


<h2>3. Live Quiz (10 questions)</h2>


Try every question first, then check the answer key in section 5.

<b>Q1.</b> Which DORA metric measures stability?
- A) Deployment frequency
- B) Lead time for changes
- C) Change failure rate
- D) Number of commits

<b>Q2.</b> True or false: an AWS budget automatically stops your resources when the limit is reached.
- A) True
- B) False

<b>Q3.</b> A team deploys 20 times in a month and 3 deployments caused incidents. What is the change failure rate?
- A) 3%
- B) 6.7%
- C) 15%
- D) 20%

<b>Q4.</b> In GitOps, what applies changes to the cluster?
- A) The developer's laptop
- B) The CI pipeline with admin credentials
- C) An in-cluster agent like Argo CD that pulls from Git
- D) The cloud provider

<b>Q5.</b> What does the "A" in CALMS stand for?
- A) Agile
- B) Automation
- C) Availability
- D) Architecture

<b>Q6.</b> An SLO is 99.9% over 30 days. Roughly how many minutes of downtime does the error budget allow?
- A) 4.3
- B) 43.2
- C) 432
- D) 0.43

<b>Q7.</b> In the shared responsibility model, who patches the operating system of an EC2 instance?
- A) AWS
- B) The customer
- C) Nobody – EC2 patches itself
- D) The Availability Zone

<b>Q8.</b> What does this command output when it works: `aws sts get-caller-identity`?
- A) A list of S3 buckets
- B) Your account ID, user ID and ARN
- C) Your monthly bill
- D) The CLI version

<b>Q9.</b> Which Azure concept is closest to an AWS account as a billing boundary?
- A) Resource group
- B) Subscription
- C) Availability Zone
- D) VNet

<b>Q10.</b> Changing a Terraform-managed resource by hand in the console creates:
- A) A pull request
- B) Drift
- C) A tag
- D) An error budget


<h2>5. Answer Key</h2>


1. C – Change failure rate (and time to restore) measure stability; the other two measure speed.
2. B (False) – Budgets only send alerts; you must delete resources yourself.
3. C – 3 ÷ 20 = 0.15 = 15%.
4. C – GitOps is pull-based; the in-cluster agent reconciles the cluster with Git.
5. B – Culture, Automation, Lean, Measurement, Sharing.
6. B – 30 × 24 × 60 × 0.001 = 43.2 minutes.
7. B – EC2 is IaaS, so the customer patches the operating system.
8. B – It returns the UserId, Account and Arn of the identity you are using.
9. B – An Azure subscription is the billing boundary, like an AWS account.
10. B – The real resource no longer matches the code: that is drift.
