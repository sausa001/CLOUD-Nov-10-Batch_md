<h1 align="left">Day 02 – Linux Administration – Questions & Answers</h1>

<p>
<img src="https://img.shields.io/badge/Week-01-1F3864?style=flat-square" alt="Week: 01">
<img src="https://img.shields.io/badge/Day-02-2E75B6?style=flat-square" alt="Day: 02">
<img src="https://img.shields.io/badge/Cloud-Local%20%2F%20EC2-FF9900?style=flat-square" alt="Cloud: Local / EC2">
<img src="https://img.shields.io/badge/Class-8%3A00%E2%80%9312%3A15%20EST-2E75B6?style=flat-square" alt="Class: 8:00–12:15 EST">
<img src="https://img.shields.io/badge/Lab-1%3A15%E2%80%935%3A00%20EST-2E7D32?style=flat-square" alt="Lab: 1:15–5:00 EST">
</p>

<p><a href="Day02_Linux_Administration.md">Notes</a> · <a href="Day02_Commands_and_Setup.md">Commands & Setup</a> · <b>Questions & Answers</b> · <a href="../README.md">Week overview</a> · <a href="../../README.md">Course index</a></p>



Companion to the main notes: [Day02_Linux_Administration.md](Day02_Linux_Administration.md). All questions from today in one place: recap, interview Q&A,
live quiz and the answer key. Use it to revise and to prepare for client interviews.


<h2>1. Stand-up Recap Questions (8:00 AM)</h2>


Answer these from memory before class starts – they review the previous day.

1. What is the difference between Continuous Delivery and Continuous Deployment?
2. Name the four DORA metrics. Which two measure stability?
3. Does an AWS budget stop your resources when you reach $25? What does it actually do?
4. What does `aws sts get-caller-identity` tell you, and why do we run it before any AWS work?
5. In one sentence: what is drift in Infrastructure as Code?


<h2>2. Interview-style Questions with Model Answers</h2>


<h3>1. "A service won't start on a Linux server. Walk me through how you troubleshoot."</h3>

<b>Model answer:</b> I run `systemctl status <svc>` for the state and exit code, then `journalctl -u <svc> -n 100 --no-pager` and read the first real error. Common causes are a wrong path in `ExecStart` (status 203/EXEC), "Permission denied" on files, a port already in use (`ss -ltnp` or `lsof -i`), or a bad config or env file. After the fix I run `daemon-reload` if I touched the unit, restart, and confirm with `status` and a health check.

<h3>2. "The disk is 100% full on a production server. What do you do?"</h3>

<b>Model answer:</b> `df -h` to find the filesystem, `df -i` to rule out inodes, then `du -xh --max-depth=1 | sort -h` to drill down. The culprit is usually `/var/log`, Docker images or `/tmp`. I truncate or compress logs rather than deleting files that are still open, check `lsof +L1`, and vacuum the journal. Then I fix the cause with logrotate or a retention policy and add a disk alarm at 80%.

<h3>3. "What does chmod 640 mean, and when would you use it?"</h3>

<b>Model answer:</b> Owner read and write, group read, others nothing. I use it for config files that the app's group must read but other users must not, for example `/etc/peoplepulse/app.env` owned by `root:peoplepulse`.

<h3>4. "Why shouldn't applications run as root?"</h3>

<b>Model answer:</b> If the app is hacked, the attacker gets full control of the host. A dedicated non-login service account with only the permissions it needs limits the damage ("blast radius"). The same rule applies in containers (`USER` in a Dockerfile) and Kubernetes (`runAsNonRoot`).

<h3>5. "What's the difference between kill and kill -9?"</h3>

<b>Model answer:</b> `kill` sends SIGTERM, which the app can catch to shut down cleanly. `kill -9` sends SIGKILL, which cannot be caught, so there is no cleanup and data can be lost. I use SIGTERM first and SIGKILL only if the process does not exit.

<h3>6. "free shows only 200 MB free. Is the server out of memory?"</h3>

<b>Model answer:</b> Not necessarily. Linux uses spare RAM for disk cache. I look at the `available` column, check swap, run `vmstat 1 5` to see if it is swapping, and search the journal for OOM-killer messages (`journalctl -k | grep -i oom`) before deciding.

<h3>7. "How do you schedule a recurring job, and what are the gotchas?"</h3>

<b>Model answer:</b> A file in `/etc/cron.d/` or a systemd timer, kept in Git. Gotchas: cron has a minimal PATH so I use full paths, the server time zone is usually UTC, I send output to a log, and I make the job safe to run twice (idempotent).

<h3>8. "How do you securely connect to an EC2 instance or Azure VM?"</h3>

<b>Model answer:</b> Key-based SSH only, private key `600`, port 22 open only to my IP in the security group or NSG. In production I prefer no public IP at all and use AWS Systems Manager Session Manager or Azure Bastion, which are audited and need no open inbound port.


<h2>3. Live Quiz (10 questions)</h2>


Try every question first, then check the answer key in section 5.

<b>Q1.</b> Which directory holds system-wide configuration files?
- A) `/var`
- B) `/etc`
- C) `/opt`
- D) `/usr`

<b>Q2.</b> What does this print? `chmod 754 run.sh && stat -c '%A' run.sh`
- A) `-rwxr-xr--`
- B) `-rwxr-x--x`
- C) `-rw-r-xr--`
- D) `-rwxrwxr--`

<b>Q3.</b> With `umask 027`, what permissions does a new <b>file</b> get?
- A) 750
- B) 640
- C) 644
- D) 600

<b>Q4.</b> True or false: `usermod -G docker haiman` adds haiman to docker and keeps Haiman's other groups.
- A) True
- B) False

<b>Q5.</b> After editing a unit file in `/etc/systemd/system/`, which command must you run before restarting?
- A) `systemctl reload-all`
- B) `systemctl daemon-reload`
- C) `journalctl --rotate`
- D) `systemctl enable`

<b>Q6.</b> What does the cron expression `*/10 9-17 * * 1-5` mean?
- A) Every 10 hours
- B) Every 10 minutes from 9:00 AM to 5:50 PM, Monday to Friday
- C) At 9:10 and 5:10 every day
- D) Every 10 minutes at weekends

<b>Q7.</b> Which command shows the process listening on port 8080?
- A) `df -h 8080`
- B) `lsof -i :8080`
- C) `free -p 8080`
- D) `du 8080`

<b>Q8.</b> A shared folder shows `drwxrws---`. What does the `s` do?
- A) Only the owner can delete files
- B) Programs inside run as root
- C) New files inherit the folder's group
- D) The folder is hidden

<b>Q9.</b> `systemctl status` shows `status=203/EXEC`. What is the most likely cause?
- A) The port is in use
- B) Out of memory
- C) The `ExecStart` program path is wrong or not executable
- D) The network is down

<b>Q10.</b> SSH says your key file permissions are "too open". Which command fixes it?
- A) `chmod 777 key.pem`
- B) `chmod 644 key.pem`
- C) `chmod 600 key.pem`
- D) `chown root key.pem`


<h2>5. Answer Key</h2>


1. B – `/etc` holds the text configuration files.
2. A – 7 = rwx, 5 = r-x, 4 = r--.
3. B – 666 minus 027 gives 640 (rw-r-----).
4. B (False) – without `-a`, it replaces Haiman's extra groups with only `docker`.
5. B – `daemon-reload` makes systemd re-read unit files.
6. B – every 10 minutes during hours 9 to 17 (last run 17:50), Monday to Friday.
7. B – `lsof -i :8080` (or `ss -ltnp`).
8. C – setgid on a directory makes new files inherit the directory's group.
9. C – 203/EXEC means systemd could not execute the `ExecStart` program.
10. C – private keys must be readable only by you: `chmod 600`.
