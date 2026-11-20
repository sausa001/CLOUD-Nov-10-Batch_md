<h1 align="left">Day 02 – Linux Administration – Commands & Setup Steps</h1>

<p>
<img src="https://img.shields.io/badge/Week-01-1F3864?style=flat-square" alt="Week: 01">
<img src="https://img.shields.io/badge/Day-02-2E75B6?style=flat-square" alt="Day: 02">
<img src="https://img.shields.io/badge/Cloud-Local%20%2F%20EC2-FF9900?style=flat-square" alt="Cloud: Local / EC2">
<img src="https://img.shields.io/badge/Class-8%3A00%E2%80%9312%3A15%20EST-2E75B6?style=flat-square" alt="Class: 8:00–12:15 EST">
<img src="https://img.shields.io/badge/Lab-1%3A15%E2%80%935%3A00%20EST-2E7D32?style=flat-square" alt="Lab: 1:15–5:00 EST">
</p>

<p><a href="Day02_Linux_Administration.md">Notes</a> · <b>Commands & Setup</b> · <a href="Day02_Questions_and_Answers.md">Questions & Answers</a> · <a href="../README.md">Week overview</a> · <a href="../../README.md">Course index</a></p>



Companion to the main notes: [Day02_Linux_Administration.md](Day02_Linux_Administration.md). Every command used today, in the order you meet it in class and lab,
with the command breakdown tables that explain each part. Run commands in your own terminal and replace
`<your-name>` with your resource name from the Batch Roster.


<h2>1. Installation & Setup Steps</h2>


Run these before or at the start of the lab (installs, logins, cluster start-up, tool configuration).

_No new installation today – you use the tools set up on earlier days._


<h2>2. Commands by Session</h2>


<h3>8:15 – 9:45 AM · Concept Session 1</h3>


<h4>2. Kernel, Shell and Distributions</h4>


```bash
cat /etc/os-release | head -3   # which distro and version
uname -r                        # kernel version
echo "$SHELL"                   # your login shell
```

Expected output:

```text
PRETTY_NAME="Ubuntu 24.04.1 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
6.8.0-1015-aws
/bin/bash
```

<h4>3. File System Hierarchy</h4>


```bash
pwd                      # where am I?
ls /                     # top-level folders
head -2 /proc/meminfo    # kernel info exposed as a "file"
```

Expected output:

```text
/home/ubuntu
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
MemTotal:         980148 kB
MemFree:          212480 kB
```

<h4>4. Essential File Commands</h4>


```bash
mkdir -p ~/day02/practice && cd ~/day02/practice
touch app.log notes.txt
cp app.log app.log.bak          # copy
mv notes.txt README.txt         # rename
ls -lah                         # long list, hidden files, human sizes
find ~/day02 -type f -name '*.log'                   # find .log files
find /var/log -type f -size +10M 2>/dev/null         # files bigger than 10 MB
find ~/day02 -type f -name '*.bak' -mtime +7 -print  # .bak files older than 7 days
cd - && rm -ri ~/day02/practice                      # back to previous dir; remove with confirmation
```

Expected output:

```text
total 8.0K
drwxrwxr-x 2 ubuntu ubuntu 4.0K Nov 10 09:50 .
drwxrwxr-x 3 ubuntu ubuntu 4.0K Nov 10 09:50 ..
-rw-rw-r-- 1 ubuntu ubuntu    0 Nov 10 09:50 README.txt
-rw-rw-r-- 1 ubuntu ubuntu    0 Nov 10 09:50 app.log
-rw-rw-r-- 1 ubuntu ubuntu    0 Nov 10 09:50 app.log.bak
/home/ubuntu/day02/practice/app.log
```

<b>Command breakdown:</b> `find ~/day02 -type f -name '*.bak' -mtime +7 -print`

| Part | What it means |
|---|---|
| `find` | search for files, starting in a folder and going down into every sub-folder |
| `~/day02` | where to start (your home folder's `day02`) |
| `-type f` | only regular files (`d` = directories) |
| `-name '*.bak'` | name matches the pattern; quote it so the shell does not expand `*` first |
| `-mtime +7` | last modified <b>more than</b> 7 days ago (`-7` = less than 7 days) |
| `-print` | show each match; swap for `-delete` only after you have checked the list |
| `2>/dev/null` (other example) | throw away error messages such as "Permission denied" |

<h4>5. Reading and Searching Files</h4>


```bash
printf 'INFO start\nERROR db timeout\nINFO request ok\nerror disk slow\n' > /tmp/app.log
head -n 2 /tmp/app.log
grep -in error /tmp/app.log          # case-insensitive, with line numbers
grep -vc INFO /tmp/app.log           # count lines that are NOT INFO
grep -E 'timeout|slow' /tmp/app.log  # either word
```

Expected output:

```text
INFO start
ERROR db timeout
2:ERROR db timeout
4:error disk slow
2
ERROR db timeout
error disk slow
```

<b>Command breakdown:</b> `journalctl -u ssh | grep -i failed | tail -5`

| Part | What it means |
|---|---|
| `journalctl -u ssh` | print all log lines of the SSH service |
| `\|` | pipe: send the output of the left command into the right command |
| `grep -i failed` | keep only lines containing "failed", ignoring upper/lower case |
| `\| tail -5` | keep only the last 5 of those lines (the most recent) |
| `grep -n` / `-v` / `-c` | add line numbers / show lines that do NOT match / only count matches |
| `grep -E 'timeout\|slow'` | extended regex: match "timeout" OR "slow" |

<h4>6. Users, Groups and sudo</h4>


```bash
sudo groupadd devops                                  # create a group
sudo useradd -m -s /bin/bash -G devops haiman          # -m home dir, -s shell, -G extra groups
sudo passwd haiman                                     # set a password (interactive)
sudo useradd --system --no-create-home --shell /usr/sbin/nologin peoplepulse  # service account, cannot log in
sudo usermod -aG devops jobanjit                           # -a = APPEND to groups (never forget -a)
id haiman                                              # show UID, GID and groups
sudo visudo -f /etc/sudoers.d/devops                  # edit sudo rules safely (syntax checked)
sudo userdel -r haiman                                 # delete user and home folder
```

Expected output:

```text
uid=1001(haiman) gid=1002(haiman) groups=1002(haiman),1001(devops)
```

<b>Command breakdown:</b> `sudo useradd -m -s /bin/bash -G devops haiman`

| Part | What it means |
|---|---|
| `sudo` | run as root – only root may create users |
| `useradd` | low-level "create a user" tool (works the same on Ubuntu, Amazon Linux and RHEL) |
| `-m` | create the home folder `/home/haiman`; without it the user has no home |
| `-s /bin/bash` | login shell; without it Ubuntu may give `/bin/sh` |
| `-G devops` | extra (supplementary) group; the group must already exist |
| `haiman` | the username |
| `--system --shell /usr/sbin/nologin` (service account) | low UID, no login possible – for apps, not people |

<b>Command breakdown:</b> `sudo usermod -aG devops jobanjit`

| Part | What it means |
|---|---|
| `usermod` | change an existing user |
| `-a` | <b>append</b> – keep the groups jobanjit already has |
| `-G devops` | the group(s) to add; without `-a` this list <i>replaces</i> all his extra groups |
| `jobanjit` | the user to change; Jobanjit must log out and in again to get the new group |

<h4>7. Permissions: rwx, Numeric, Symbolic and Ownership</h4>


```bash
chmod 640 /etc/peoplepulse/app.env               # numeric
chmod u+x deploy.sh                              # symbolic: add execute for owner
chmod g-w,o-rwx report.csv                       # remove group write, remove all for others
chmod -R g+rX /opt/peoplepulse                   # capital X = execute only on dirs (and already-executable files)
sudo chown peoplepulse:devops /opt/peoplepulse/data    # change owner and group
sudo chown -R peoplepulse: /var/log/peoplepulse        # "user:" = user + that user's primary group
sudo chgrp devops /srv/shared                    # change group only
stat -c '%a %U:%G %n' /etc/peoplepulse/app.env   # show numeric mode, owner, group
```

Expected output:

```text
640 root:peoplepulse /etc/peoplepulse/app.env
```

<b>Command breakdown:</b> `chmod -R g+rX /opt/peoplepulse`

| Part | What it means |
|---|---|
| `chmod` | change mode (permissions) |
| `-R` | recursive – apply to the folder and everything inside it |
| `g` | who: the group (`u` owner, `o` others, `a` all) |
| `+` | add (`-` remove, `=` set exactly) |
| `r` | read |
| `X` (capital) | execute only for directories and files that are already executable – safe for mixed trees |
| `/opt/peoplepulse` | target; a wrong path with `-R` (for example `/`) can break the whole server |

<b>Command breakdown:</b> `sudo chown -R peoplepulse: /var/log/peoplepulse`

| Part | What it means |
|---|---|
| `chown` | change owner (and optionally group) |
| `-R` | recursive – every file and folder inside |
| `peoplepulse:` | new owner `peoplepulse`; the colon with nothing after it sets that user's primary group too |
| `peoplepulse:devops` (other example) | owner `peoplepulse`, group `devops` |
| `/var/log/peoplepulse` | target folder |

<h4>8. Special Bits: setuid, setgid and Sticky</h4>


```bash
sudo mkdir -p /srv/devops-shared
sudo chown root:devops /srv/devops-shared
sudo chmod 2770 /srv/devops-shared   # 2 = setgid, 770 = rwx for owner and group, nothing for others
ls -ld /srv/devops-shared /tmp
```

Expected output:

```text
drwxrws--- 2 root devops 4096 Nov 10 10:33 /srv/devops-shared
drwxrwxrwt 9 root root   4096 Nov 10 10:30 /tmp
```

<b>Command breakdown:</b> `sudo chmod 2770 /srv/devops-shared`

| Part | What it means |
|---|---|
| `2` (first of four digits) | special bit: 4 = setuid, 2 = setgid, 1 = sticky; here setgid |
| `7` | owner (root): rwx |
| `7` | group (devops): rwx – members can create and edit files |
| `0` | others: no access at all |
| `/srv/devops-shared` | the shared folder; `ls -ld` shows `drwxrws---` (the `s` is setgid) |

<h4>9. umask – Default Permissions for New Files</h4>


```bash
umask              # show the current value
umask 027          # change it for this shell only
touch a.txt; mkdir d
ls -ld a.txt d
```

Expected output:

```text
0002
-rw-r----- 1 ubuntu ubuntu    0 Nov 10 10:40 a.txt
drwxr-x--- 2 ubuntu ubuntu 4096 Nov 10 10:40 d
```

<h3>10:00 – 11:15 AM · Concept Session 2 + Live Demo</h3>


<h4>10. Processes and Signals</h4>


```bash
ps aux | head -3                     # all processes: USER PID %CPU %MEM ... COMMAND
ps -ef --forest | less               # parent/child tree
pgrep -a python3                     # find PIDs by name
top                                  # live view: P = sort by CPU, M = by memory, q = quit
sudo apt-get install -y htop && htop # friendlier top (F9 = kill, F10 = quit)
uptime                               # load average for 1, 5, 15 minutes
nproc                                # number of CPUs
kill 4821                            # SIGTERM: please stop cleanly
kill -9 4821                         # SIGKILL: last resort only
```

Expected output:

```text
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  1.3  22064 13180 ?        Ss   09:01   0:02 /sbin/init
root           2  0.0  0.0      0     0 ?        S    09:01   0:00 [kthreadd]
 10:59:12 up  1:58,  1 user,  load average: 0.52, 0.40, 0.31
2
```

<b>Command breakdown:</b> `ps aux | head -3`

| Part | What it means |
|---|---|
| `ps` | process status – a snapshot of processes |
| `a` | processes of all users |
| `u` | user-friendly columns: USER, %CPU, %MEM, START, COMMAND |
| `x` | include processes with no terminal (services/daemons) |
| `\| head -3` | keep only the header and first two lines |
| `STAT` column | `R` running, `S` sleeping, `Z` zombie, `s` session leader |

<b>Command breakdown:</b> `kill -9 4821`

| Part | What it means |
|---|---|
| `kill` | send a signal to a process (it does not always "kill") |
| `-9` | which signal: 9 = SIGKILL; default with no number is 15 = SIGTERM; `-HUP` or `-1` = reload |
| `4821` | the PID; a wrong PID stops the wrong program, so check with `ps` or `pgrep -a` first |

<h4>11. systemd Services</h4>


<b>Command breakdown:</b> `sudo systemctl enable --now peoplepulse-stub.service`

| Part | What it means |
|---|---|
| `sudo` | changing services needs root (reading `status` does not) |
| `systemctl` | the command-line tool to talk to systemd |
| `enable` | create a link so the service starts at every boot |
| `--now` | also start it right now (otherwise `enable` alone waits for the next reboot) |
| `peoplepulse-stub.service` | unit name; `.service` can be left out; a typo gives "Unit ... not found" |

<h4>12. journalctl – Reading Service Logs</h4>


<b>Command breakdown:</b> `journalctl -u peoplepulse-stub --since "10 min ago" -n 50 --no-pager`

| Part | What it means |
|---|---|
| `journalctl` | read the systemd journal |
| `-u peoplepulse-stub` | only this unit; without `-u` you get every log on the server |
| `--since "10 min ago"` | time filter; also accepts `"2026-11-10 09:00"`, `today`, `yesterday` |
| `-n 50` | only the last 50 lines |
| `--no-pager` | print straight to the screen instead of opening `less` (good for scripts and copy/paste) |
| `-f` (other example) | follow new lines live; Ctrl+C to stop |
| `-p err -b` (other example) | priority "error" and worse, since the last boot |

```bash
journalctl -u ssh -n 3 --no-pager     # on RHEL/Amazon Linux the unit is sshd
```

Expected output:

```text
Nov 10 11:20:02 ip-172-31-5-10 sshd[1520]: Accepted publickey for ubuntu from 203.0.113.25 port 52144 ssh2: ED25519 SHA256:abc...
Nov 10 11:20:02 ip-172-31-5-10 sshd[1520]: pam_unix(sshd:session): session opened for user ubuntu(uid=1000) by ubuntu(uid=0)
Nov 10 11:21:40 ip-172-31-5-10 sshd[1533]: Invalid user admin from 198.51.100.7 port 41220
```

<h4>13. cron – Scheduled Jobs</h4>


```bash
crontab -l                       # list your jobs
crontab -e                       # edit your jobs (opens an editor)
# Add this line in the editor: every day at 01:30, save disk usage to a file
# 30 1 * * * /usr/bin/df -h / >> /home/ubuntu/disk-report.log 2>&1
```

Expected output:

```text
no crontab for ubuntu
```

<h4>14. Package Managers: apt and dnf</h4>


```bash
sudo apt-get update && sudo apt-get install -y tree
tree -L 1 /etc/systemd
dpkg -S /usr/bin/tree
```

Expected output:

```text
/etc/systemd
├── journald.conf
├── logind.conf
├── system
├── system.conf
└── user
...
tree: /usr/bin/tree
```

<b>Command breakdown:</b> `sudo apt-get update && sudo apt-get install -y tree`

| Part | What it means |
|---|---|
| `sudo apt-get update` | download the latest package <b>list</b> (it does not upgrade anything) |
| `&&` | run the next command only if the first one succeeded |
| `apt-get install` | download and install the package and its dependencies |
| `-y` | answer "yes" automatically; without it a script or Dockerfile stops at the prompt |
| `tree` | package name |
| `sudo dnf install -y tree` (Amazon Linux/RHEL) | same job; dnf refreshes its list automatically |

<h4>15. Disk and Memory</h4>


```bash
df -h /
free -h
```

Expected output:

```text
Filesystem      Size  Used Avail Use% Mounted on
/dev/root       7.6G  2.1G  5.5G  28% /
               total        used        free      shared  buff/cache   available
Mem:           957Mi       410Mi       112Mi       1.0Mi       590Mi       547Mi
Swap:             0B          0B          0B
```

<b>Command breakdown:</b> `sudo du -xh / --max-depth=1 2>/dev/null | sort -h | tail`

| Part | What it means |
|---|---|
| `sudo` | read every folder, including ones only root can open |
| `du` | disk usage – add up the size of files in folders |
| `-x` | stay on this one filesystem (skip `/proc` and other mounted disks) |
| `-h` | human sizes (K, M, G) |
| `/` | start at the root |
| `--max-depth=1` | show totals only for the top-level folders, not every sub-folder |
| `2>/dev/null` | hide "Permission denied" and similar errors |
| `\| sort -h` | sort by human-readable size, smallest first |
| `\| tail` | show the last 10 lines = the 10 biggest folders |

<b>Command breakdown:</b> `sudo lsof -i :8080` and `sudo lsof +L1`

| Part | What it means |
|---|---|
| `lsof` | list open files – in Linux, network sockets are files too |
| `sudo` | without it you only see your own processes |
| `-i :8080` | only network connections on port 8080; shows COMMAND, PID and USER |
| `+L1` | files with fewer than 1 link, i.e. deleted but still held open – the hidden cause of "disk still full" |

<h4>16. SSH Basics: Connecting to a Cloud VM</h4>


```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -C "<your-name>@laptop"  # create a key pair (once)
chmod 600 ~/.ssh/day02-<your-name>.pem                             # private key must be private
ssh -i ~/.ssh/day02-<your-name>.pem ubuntu@<PUBLIC_IP>             # connect
scp -i ~/.ssh/day02-<your-name>.pem notes.txt ubuntu@<PUBLIC_IP>:/tmp/   # copy a file to the VM
```

Expected output:

```text
The authenticity of host '<PUBLIC_IP> (<PUBLIC_IP>)' can't be established.
ED25519 key fingerprint is SHA256:Xq3f...
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Welcome to Ubuntu 24.04.1 LTS (GNU/Linux 6.8.0-1015-aws x86_64)
ubuntu@ip-172-31-5-10:~$
```

<b>Command breakdown:</b> `ssh -i ~/.ssh/day02-<your-name>.pem ubuntu@<PUBLIC_IP>`

| Part | What it means |
|---|---|
| `ssh` | open a secure remote shell |
| `-i ~/.ssh/day02-<your-name>.pem` | identity file = which private key to use; must be mode `600` |
| `ubuntu` | remote username; wrong user (for example `root` or `ec2-user` on Ubuntu) gives "Permission denied (publickey)" |
| `@<PUBLIC_IP>` | the VM's public IP or DNS name |
| `-p 2222` (optional) | use a different port than 22 |
| `-v` (optional) | verbose – shows each step, very useful when login fails |

<b>Command breakdown:</b> `scp -i key.pem notes.txt ubuntu@<PUBLIC_IP>:/tmp/`

| Part | What it means |
|---|---|
| `scp` | secure copy over SSH |
| `-i key.pem` | same private key as for `ssh` |
| `notes.txt` | local source file (use `-r` for a folder) |
| `ubuntu@<PUBLIC_IP>:/tmp/` | destination: user, host, then `:` and the remote path; swap the order to download |

<h4>Live Demo: Run the PeoplePulse Stub as a systemd Service</h4>


```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin peoplepulse
sudo mkdir -p /opt/peoplepulse/www /var/log/peoplepulse /etc/peoplepulse
echo '{"status":"ok","app":"peoplepulse-stub"}' | sudo tee /opt/peoplepulse/www/health > /dev/null
sudo chown -R root:peoplepulse /opt/peoplepulse
sudo chmod -R 750 /opt/peoplepulse
sudo chown peoplepulse:peoplepulse /var/log/peoplepulse
sudo chmod 750 /var/log/peoplepulse
echo 'PORT=8080' | sudo tee /etc/peoplepulse/app.env > /dev/null
sudo chown root:peoplepulse /etc/peoplepulse/app.env && sudo chmod 640 /etc/peoplepulse/app.env
ls -l /opt/peoplepulse /etc/peoplepulse
```

Expected output:

```text
/etc/peoplepulse:
total 4
-rw-r----- 1 root peoplepulse 10 Nov 10 11:56 app.env

/opt/peoplepulse:
total 4
drwxr-x--- 2 root peoplepulse 4096 Nov 10 11:56 www
```

<b>Command breakdown:</b> `echo 'PORT=8080' | sudo tee /etc/peoplepulse/app.env > /dev/null`

| Part | What it means |
|---|---|
| `echo 'PORT=8080'` | print the text we want in the file |
| `\|` | pipe the text into the next command |
| `sudo tee /etc/peoplepulse/app.env` | write it to a root-owned file; `sudo echo ... > file` fails because the `>` runs as you, not root |
| `> /dev/null` | hide the copy that `tee` also prints to the screen |
| `tee -a` (other use) | append instead of overwrite |

```bash
sudo tee /etc/systemd/system/peoplepulse-stub.service > /dev/null <<'EOF'
[Unit]
Description=PeoplePulse stub web service
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=peoplepulse
Group=peoplepulse
EnvironmentFile=/etc/peoplepulse/app.env
WorkingDirectory=/opt/peoplepulse
ExecStart=/usr/bin/python3 -m http.server ${PORT} --bind 127.0.0.1 --directory /opt/peoplepulse/www
Restart=on-failure
RestartSec=5
UMask=0027
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
EOF
```

<b>Command breakdown:</b> the `ExecStart=` line and its neighbours

| Part | What it means |
|---|---|
| `<<'EOF' ... EOF` | here-document: everything between is written to the file; quotes on `'EOF'` stop the shell from expanding `${PORT}` |
| `/usr/bin/python3` | full path – systemd does not search `PATH` the way your shell does |
| `-m http.server` | run Python's built-in web server module |
| `${PORT}` | replaced by systemd with the value from `EnvironmentFile` (8080) |
| `--bind 127.0.0.1` | listen only on this machine, not on the network |
| `--directory /opt/peoplepulse/www` | folder to serve files from |
| `User=` / `Group=` | run as the service account, never root |
| `UMask=0027` | new files the app creates get 640 / 750 |
| `NoNewPrivileges=true` | the process can never gain more rights (for example through setuid programs) |

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now peoplepulse-stub.service
systemctl status peoplepulse-stub --no-pager
```

Expected output:

```text
● peoplepulse-stub.service - PeoplePulse stub web service
     Loaded: loaded (/etc/systemd/system/peoplepulse-stub.service; enabled; preset: enabled)
     Active: active (running) since Tue 2026-11-10 12:00:04 EST; 3s ago
   Main PID: 4821 (python3)
     CGroup: /system.slice/peoplepulse-stub.service
             └─4821 /usr/bin/python3 -m http.server 8080 --bind 127.0.0.1 --directory /opt/peoplepulse/www
```

```bash
curl -s http://127.0.0.1:8080/health
ps -o user,pid,cmd -C python3
sudo lsof -i :8080
```

Expected output:

```text
{"status":"ok","app":"peoplepulse-stub"}
USER         PID CMD
peoplep+    4821 /usr/bin/python3 -m http.server 8080 --bind 127.0.0.1 --directory /opt/peoplepulse/www
COMMAND  PID        USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
python3 4821 peoplepulse    3u  IPv4  51234      0t0  TCP localhost:http-alt (LISTEN)
```

```bash
sudo kill -9 "$(systemctl show -p MainPID --value peoplepulse-stub)"
sleep 6
systemctl status peoplepulse-stub --no-pager | head -3
journalctl -u peoplepulse-stub -n 5 --no-pager
```

Expected output:

```text
     Active: active (running) since Tue 2026-11-10 12:02:15 EST; 1s ago
... peoplepulse-stub.service: Main process exited, code=killed, status=9/KILL
... peoplepulse-stub.service: Failed with result 'signal'.
... peoplepulse-stub.service: Scheduled restart job, restart counter is at 1.
... Started peoplepulse-stub.service - PeoplePulse stub web service.
```

<b>Command breakdown:</b> `sudo kill -9 "$(systemctl show -p MainPID --value peoplepulse-stub)"`

| Part | What it means |
|---|---|
| `$( ... )` | command substitution: run the inner command and paste its output here |
| `systemctl show -p MainPID` | ask systemd for one property of the unit: its main process ID |
| `--value` | print only the number (`4821`), not `MainPID=4821` |
| `kill -9` | send SIGKILL – simulates a crash, so `Restart=on-failure` kicks in (a normal `systemctl stop` would not restart) |
| `"..."` | quotes keep the result as one word |

```bash
df -h / && free -h && uptime
```

<h3>1:15 – 2:45 PM · Lab Part 1 – Task Assignment & Guided Build</h3>


<h4>Task 0 – Prepare Your Linux Machine</h4>


```bash
ps -p 1 -o comm=      # must print: systemd
sudo -v && echo "sudo OK"
```

```bash
export AWS_REGION=us-east-1
MY_IP=$(curl -s https://checkip.amazonaws.com)
AMI_ID=$(aws ssm get-parameter \
  --name /aws/service/canonical/ubuntu/server/24.04/stable/current/amd64/hvm/ebs-gp3/ami-id \
  --query Parameter.Value --output text)

aws ec2 create-key-pair --key-name day02-<your-name> --key-type ed25519 \
  --query KeyMaterial --output text > ~/.ssh/day02-<your-name>.pem
chmod 600 ~/.ssh/day02-<your-name>.pem

VPC_ID=$(aws ec2 describe-vpcs --filters Name=is-default,Values=true --query 'Vpcs[0].VpcId' --output text)
SG_ID=$(aws ec2 create-security-group --group-name day02-ssh-<your-name> \
  --description "Day02 SSH from my IP" --vpc-id "$VPC_ID" --query GroupId --output text)
aws ec2 authorize-security-group-ingress --group-id "$SG_ID" --protocol tcp --port 22 --cidr "${MY_IP}/32"

INSTANCE_ID=$(aws ec2 run-instances --image-id "$AMI_ID" --instance-type t3.micro \
  --key-name day02-<your-name> --security-group-ids "$SG_ID" \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=day02-linux},{Key=owner,Value=<your-name>},{Key=env,Value=dev},{Key=course,Value=devops-8wk}]' \
  --query 'Instances[0].InstanceId' --output text)
aws ec2 wait instance-running --instance-ids "$INSTANCE_ID"
PUBLIC_IP=$(aws ec2 describe-instances --instance-ids "$INSTANCE_ID" \
  --query 'Reservations[0].Instances[0].PublicIpAddress' --output text)
echo "$INSTANCE_ID $SG_ID $PUBLIC_IP" | tee ~/day02-ids.txt
ssh -i ~/.ssh/day02-<your-name>.pem ubuntu@"$PUBLIC_IP"
```

<b>Command breakdown:</b> `aws ec2 run-instances ...`

| Part | What it means |
|---|---|
| `aws ec2 run-instances` | AWS CLI v2: launch one or more EC2 instances |
| `--image-id "$AMI_ID"` | which OS image (AMI); we looked up the latest Ubuntu 24.04 from a public SSM parameter, so no hard-coded ID |
| `--instance-type t3.micro` | size: 2 vCPU, 1 GiB RAM – small and cheap; a typo like `t3.xlarge` costs about 16 times more |
| `--key-name day02-<your-name>` | which key pair's public key AWS puts in `authorized_keys` |
| `--security-group-ids "$SG_ID"` | firewall that allows port 22 only from your IP |
| `--tag-specifications '...'` | tags `owner`, `env`, `course` so you can find and clean up (and track cost) |
| `--query 'Instances[0].InstanceId'` | JMESPath filter: return only the instance ID |
| `--output text` | plain text, easy to store in a shell variable |
| `aws ec2 wait instance-running` | pause until the instance is running before asking for its IP |

```bash
MY_IP=$(curl -s https://checkip.amazonaws.com)
RG=rg-day02-<your-name>
az group create --name "$RG" --location eastus \
  --tags owner=<your-name> env=dev course=devops-8wk
az vm create --resource-group "$RG" --name day02-linux \
  --image Ubuntu2404 --size Standard_B1s \
  --admin-username azureuser --generate-ssh-keys \
  --public-ip-sku Standard \
  --tags owner=<your-name> env=dev course=devops-8wk
# az vm create opens SSH to any address by default - limit it to your IP
az network nsg rule update --resource-group "$RG" --nsg-name day02-linuxNSG \
  --name default-allow-ssh --source-address-prefixes "${MY_IP}/32"
PUBLIC_IP=$(az vm show -d --resource-group "$RG" --name day02-linux --query publicIps -o tsv)
ssh azureuser@"$PUBLIC_IP"
```

<b>Command breakdown:</b> `az vm create ...`

| Part | What it means |
|---|---|
| `az vm create` | Azure CLI: create a VM plus the network pieces it needs (VNet, NIC, public IP, NSG) |
| `--resource-group "$RG"` | the folder that holds everything; deleting it deletes all of it |
| `--name day02-linux` | VM name; Azure names the firewall `day02-linuxNSG` |
| `--image Ubuntu2404` | short alias for the Ubuntu 24.04 LTS image |
| `--size Standard_B1s` | 1 vCPU, 1 GiB RAM burstable – the cheap lab size |
| `--admin-username azureuser` | the Linux user you log in as |
| `--generate-ssh-keys` | use your existing `~/.ssh/id_rsa.pub`, or create a key pair if none exists |
| `--public-ip-sku Standard` | the current public IP type (Basic is retired) |
| `--tags ...` | same `owner`, `env`, `course` tags as on AWS |
| `az network nsg rule update ... --source-address-prefixes` | tighten the default SSH rule from "any" to your IP only |

<h4>Task 1 – Users, Groups and a Shared Team Folder</h4>


```bash
# One Linux account per trainee in the Nov-10 batch
sudo groupadd devops                               # platform team: owns the shared runbooks
sudo groupadd qa                                   # QA team: must NOT read devops files
for u in haiman cindia jobanjit; do sudo useradd -m -s /bin/bash -G devops "$u"; done
for u in carla raymundo;         do sudo useradd -m -s /bin/bash -G qa     "$u"; done
for u in haiman cindia jobanjit carla raymundo; do
  echo "$u:Lab-Pass-123!" | sudo chpasswd          # lab-only passwords
done

sudo mkdir -p /srv/devops-shared
sudo chown root:devops /srv/devops-shared
sudo chmod 2770 /srv/devops-shared

sudo -u haiman bash -c 'umask 007; echo "runbook v1" > /srv/devops-shared/runbook.txt'
sudo ls -l /srv/devops-shared
sudo -u jobanjit   cat /srv/devops-shared/runbook.txt     # works
sudo -u carla cat /srv/devops-shared/runbook.txt     # should fail
id haiman; id carla
```

Expected output:

```text
total 4
-rw-rw---- 1 haiman devops 11 Nov 10 14:22 runbook.txt
runbook v1
cat: /srv/devops-shared/runbook.txt: Permission denied
uid=1001(haiman) gid=1001(haiman) groups=1001(haiman),1100(devops)
uid=1004(carla) gid=1004(carla) groups=1004(carla),1101(qa)
```

<h4>Task 2 – Limited sudo for the devops Group</h4>


```bash
echo '%devops ALL=(root) NOPASSWD: /usr/bin/systemctl restart peoplepulse-stub.service, /usr/bin/systemctl status peoplepulse-stub.service' \
  | sudo tee /etc/sudoers.d/devops > /dev/null
sudo chmod 440 /etc/sudoers.d/devops
sudo visudo -c                                  # syntax check
sudo -l -U haiman                                # what may haiman run?
```

Expected output:

```text
/etc/sudoers: parsed OK
/etc/sudoers.d/devops: parsed OK
User haiman may run the following commands on day02-linux:
    (root) NOPASSWD: /usr/bin/systemctl restart peoplepulse-stub.service, /usr/bin/systemctl status peoplepulse-stub.service
```

<b>Command breakdown:</b> the sudoers rule `%devops ALL=(root) NOPASSWD: /usr/bin/systemctl restart peoplepulse-stub.service`

| Part | What it means |
|---|---|
| `%devops` | applies to the group `devops` (`%` means group; without it, a user name) |
| `ALL` | on any host name (this file is per server anyway) |
| `(root)` | may run the command as root |
| `NOPASSWD:` | no password prompt – needed for automation; leave it out for humans if policy requires |
| `/usr/bin/systemctl restart peoplepulse-stub.service` | the <b>exact</b> allowed command with full path; anything else is refused |
| `chmod 440` | sudoers files must be read-only, or sudo ignores them |
| `visudo -c` | check the syntax of all sudoers files before you log out |

<h4>Task 3 – Run the PeoplePulse Stub with systemd</h4>


```bash
sudo mkdir -p /etc/systemd/system/peoplepulse-stub.service.d
sudo tee /etc/systemd/system/peoplepulse-stub.service.d/logging.conf > /dev/null <<'EOF'
[Service]
StandardError=append:/var/log/peoplepulse/access.log
EOF
sudo systemctl daemon-reload && sudo systemctl restart peoplepulse-stub
for i in 1 2 3; do curl -s 127.0.0.1:8080/health > /dev/null; done
sudo tail -3 /var/log/peoplepulse/access.log
systemctl cat peoplepulse-stub | grep -A1 logging.conf
sudo -u haiman sudo systemctl restart peoplepulse-stub.service   # allowed by Task 2
```

Expected output:

```text
127.0.0.1 - - [10/Nov/2026 14:51:02] "GET /health HTTP/1.1" 200 -
127.0.0.1 - - [10/Nov/2026 14:51:02] "GET /health HTTP/1.1" 200 -
127.0.0.1 - - [10/Nov/2026 14:51:02] "GET /health HTTP/1.1" 200 -
# /etc/systemd/system/peoplepulse-stub.service.d/logging.conf
[Service]
```

<h4>Task 4 – Schedule a Log-Cleanup cron Job</h4>


```bash
for d in 3 8 15 30; do
  sudo -u peoplepulse touch -d "$d days ago" /var/log/peoplepulse/app-$d.log
done
sudo ls -l --time-style=+%F /var/log/peoplepulse/
```

```bash
sudo tee /usr/local/bin/peoplepulse-logclean.sh > /dev/null <<'EOF'
#!/bin/bash
# Delete PeoplePulse *.log files older than RETENTION_DAYS (default 7).
set -euo pipefail
LOG_DIR="/var/log/peoplepulse"
RETENTION_DAYS="${RETENTION_DAYS:-7}"
echo "$(date -Is) cleanup start dir=${LOG_DIR} retention=${RETENTION_DAYS}d"
/usr/bin/find "$LOG_DIR" -maxdepth 1 -type f -name '*.log' -mtime +"$RETENTION_DAYS" -print -delete
echo "$(date -Is) cleanup done"
EOF
sudo chmod 755 /usr/local/bin/peoplepulse-logclean.sh
```

<b>Command breakdown:</b> the cleanup line `/usr/bin/find "$LOG_DIR" -maxdepth 1 -type f -name '*.log' -mtime +"$RETENTION_DAYS" -print -delete`

| Part | What it means |
|---|---|
| `set -euo pipefail` (top of script) | stop on any error, on unset variables and on failures inside pipes |
| `${RETENTION_DAYS:-7}` | use the variable if set, otherwise default to 7 |
| `/usr/bin/find` | full path, because cron has a tiny `PATH` |
| `"$LOG_DIR"` | the folder; quoted so spaces cannot break it; an empty value would be dangerous, which `set -u` prevents |
| `-maxdepth 1` | only this folder, not sub-folders |
| `-type f -name '*.log'` | only regular files ending in `.log` – `access.log` is safe because it is new |
| `-mtime +"$RETENTION_DAYS"` | modified more than 7 days ago |
| `-print -delete` | print each file (for the log), then delete it |

```bash
sudo /usr/local/bin/peoplepulse-logclean.sh
sudo ls /var/log/peoplepulse/
```

Expected output:

```text
2026-11-10T15:10:11-05:00 cleanup start dir=/var/log/peoplepulse retention=7d
/var/log/peoplepulse/app-8.log
/var/log/peoplepulse/app-15.log
/var/log/peoplepulse/app-30.log
2026-11-10T15:10:11-05:00 cleanup done
access.log  app-3.log
```

```bash
sudo tee /etc/cron.d/peoplepulse-logclean > /dev/null <<'EOF'
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
# m h dom mon dow user command
0 2 * * * root /usr/local/bin/peoplepulse-logclean.sh >> /var/log/peoplepulse-logclean.log 2>&1
EOF
sudo chmod 644 /etc/cron.d/peoplepulse-logclean
timedatectl | grep "Time zone"
```

<b>Command breakdown:</b> the cron line `0 2 * * * root /usr/local/bin/peoplepulse-logclean.sh >> /var/log/peoplepulse-logclean.log 2>&1`

| Part | What it means |
|---|---|
| `0` | minute 0 |
| `2` | hour 2 (02:00 in the <b>server's</b> time zone, usually UTC on cloud VMs) |
| `*` (day of month) | every day of the month |
| `*` (month) | every month |
| `*` (day of week) | every day of the week |
| `root` | user to run as – only in `/etc/cron.d` files, never in `crontab -e` |
| `/usr/local/bin/peoplepulse-logclean.sh` | the command, with full path |
| `>> /var/log/peoplepulse-logclean.log` | append normal output to a log file (`>` would overwrite it every night) |
| `2>&1` | send errors (stream 2) to the same place as normal output (stream 1) |
| `SHELL=` / `PATH=` lines | set a known shell and search path for every job in the file |

```bash
sudo sed -i 's/^0 2 \* \* \*/* * * * */' /etc/cron.d/peoplepulse-logclean
sudo -u peoplepulse touch -d "10 days ago" /var/log/peoplepulse/app-10.log
sleep 70
sudo tail -3 /var/log/peoplepulse-logclean.log
grep CRON /var/log/syslog | tail -2          # Amazon Linux/RHEL: sudo journalctl -u crond -n 5
sudo sed -i 's/^\* \* \* \* \*/0 2 * * */' /etc/cron.d/peoplepulse-logclean
grep -v '^#' /etc/cron.d/peoplepulse-logclean | tail -1
```

Expected output:

```text
2026-11-10T15:14:01-05:00 cleanup start dir=/var/log/peoplepulse retention=7d
/var/log/peoplepulse/app-10.log
2026-11-10T15:14:01-05:00 cleanup done
Nov 10 15:14:01 day02-linux CRON[6120]: (root) CMD (/usr/local/bin/peoplepulse-logclean.sh >> /var/log/peoplepulse-logclean.log 2>&1)
0 2 * * * root /usr/local/bin/peoplepulse-logclean.sh >> /var/log/peoplepulse-logclean.log 2>&1
```

<h3>2:45 – 4:00 PM · Lab Part 2 – Independent Challenge</h3>


<h4>Challenge 1 – Break/Fix: Troubleshoot a Failing Service</h4>


```bash
sudo tee /etc/systemd/system/pp-broken.service > /dev/null <<'EOF'
[Unit]
Description=PeoplePulse broken copy (troubleshooting exercise)

[Service]
User=peoplepulse
ExecStart=/usr/bin/python -m http.server 80 --directory /opt/peoplepulse/www
Restart=on-failure
RestartSec=3
StartLimitBurst=3

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl start pp-broken
```

<h4>Challenge 2 – Who Filled the Disk?</h4>


```bash
sudo fallocate -l 1G /var/tmp/.cache-blob
sudo -u peoplepulse bash -c 'fallocate -l 300M /var/log/peoplepulse/debug.log'
```

<h3>4:00 – 4:45 PM · Lab Review with Trainer & Teardown</h3>


<h4>Save Your Files First</h4>


```bash
mkdir -p ~/devops-8wk/devops-8wk-<your-name>/day02
cd ~/devops-8wk/devops-8wk-<your-name>/day02
cp /etc/systemd/system/peoplepulse-stub.service .
cp /etc/cron.d/peoplepulse-logclean .
cp ~/worksheet.md ./worksheet.md
```

<h4>Teardown Commands</h4>


```bash
sudo systemctl disable --now pp-broken peoplepulse-stub
sudo rm -rf /etc/systemd/system/pp-broken.service /etc/systemd/system/peoplepulse-stub.service \
  /etc/systemd/system/peoplepulse-stub.service.d
sudo systemctl daemon-reload && sudo systemctl reset-failed
sudo rm -f /etc/cron.d/peoplepulse-logclean /usr/local/bin/peoplepulse-logclean.sh /etc/sudoers.d/devops
for u in haiman cindia jobanjit carla raymundo; do sudo userdel -r "$u"; done
sudo userdel peoplepulse; sudo groupdel devops; sudo groupdel qa
sudo rm -rf /opt/peoplepulse /etc/peoplepulse /var/log/peoplepulse /var/log/peoplepulse-logclean.log \
  /srv/devops-shared /var/tmp/.cache-blob
```

```bash
read INSTANCE_ID SG_ID PUBLIC_IP < ~/day02-ids.txt
aws ec2 terminate-instances --instance-ids "$INSTANCE_ID"
aws ec2 wait instance-terminated --instance-ids "$INSTANCE_ID"
aws ec2 delete-security-group --group-id "$SG_ID"
aws ec2 delete-key-pair --key-name day02-<your-name>
rm -f ~/.ssh/day02-<your-name>.pem ~/day02-ids.txt
```

```bash
az group delete --name rg-day02-<your-name> --yes
```

<h4>Verify Nothing Is Billing</h4>


```bash
# AWS: expect empty output
aws ec2 describe-instances \
  --filters Name=tag:course,Values=devops-8wk Name=instance-state-name,Values=pending,running,stopping,stopped \
  --query 'Reservations[].Instances[].InstanceId' --output text
aws ec2 describe-volumes --filters Name=status,Values=available \
  --query 'Volumes[].VolumeId' --output text

# Azure: expect an empty list
az group list --tag course=devops-8wk -o table
```

<h3>4:45 – 5:00 PM · Commit & Daily Report</h3>


```bash
cd ~/devops-8wk/devops-8wk-<your-name>
git switch main && git pull
git switch -c day02-linux
git add day02
git commit -m "docs(day02): linux lab worksheet, systemd unit and log-cleanup cron"
git push -u origin day02-linux
```


<h2>3. Cheat Sheet</h2>


| Area | Command | Purpose |
|---|---|---|
| System | `cat /etc/os-release`, `uname -r` | distro and kernel version |
| Files | `ls -lah`, `cd -`, `cp -r`, `mv -i`, `rm -ri`, `mkdir -p` | everyday file work |
| Files | `find /var/log -type f -name '*.log' -mtime +7 -print` | find old logs (add `-delete` after checking) |
| Read | `less f`, `head -n 20 f`, `tail -f f` | view files and follow logs |
| Search | `grep -in error f`, `grep -r word dir`, `grep -E 'a\|b' f` | find lines |
| Users | `sudo useradd -m -s /bin/bash -G devops haiman` | create user with home + group |
| Users | `sudo usermod -aG devops jobanjit` / `id jobanjit` | append to group / check |
| Users | `sudo useradd --system --shell /usr/sbin/nologin svc` | service account |
| sudo | `sudo visudo -f /etc/sudoers.d/x`, `sudo -l -U haiman` | edit rules safely / list rights |
| Perms | `chmod 640 f` / `chmod u+x f` / `chmod 2770 dir` | numeric / symbolic / setgid |
| Perms | `sudo chown user:group f` / `stat -c '%a %U:%G' f` | ownership / check |
| Perms | `umask 027`, `namei -l /path/file` | defaults / debug a path |
| Process | `ps aux`, `pgrep -a name`, `top`, `htop`, `uptime` | inspect |
| Process | `kill PID` then `kill -9 PID` | stop politely, then force |
| systemd | `systemctl daemon-reload` | reload unit files |
| systemd | `systemctl enable --now svc` / `status svc` | start now + at boot / state |
| systemd | `systemctl list-units --type=service --state=failed` | failed services |
| Logs | `journalctl -u svc -n 50 --no-pager` / `-f` | recent / follow |
| Logs | `journalctl -p err -b`, `journalctl --disk-usage` | errors since boot / journal size |
| cron | `crontab -e` / `/etc/cron.d/<file>` (with user field) | schedule jobs |
| Packages | `sudo apt-get update && sudo apt-get install -y x` / `sudo dnf install -y x` | install |
| Disk | `df -h`, `df -i`, `du -sh *`, `du -xh / --max-depth=1 \| sort -h` | space / inodes / biggest dirs |
| Memory | `free -h`, `vmstat 1 5` | RAM ("available") |
| Open files | `sudo lsof -i :8080`, `sudo lsof +L1` | port owner / deleted-open files |
| SSH | `ssh -i key.pem ubuntu@IP`, `scp -i key.pem f ubuntu@IP:/tmp/` | connect / copy |

```text
Permissions: r=4 w=2 x=1    755 rwxr-xr-x   644 rw-r--r--   640 rw-r-----   600 rw-------
Special:     4000 setuid    2000 setgid (dir: inherit group)    1000 sticky (dir: owner-only delete)
umask:       022 -> 644/755   027 -> 640/750   077 -> 600/700
cron:        m h dom mon dow   "0 2 * * *" = daily 02:00   "*/15 * * * *" = every 15 min
Signals:     15 SIGTERM (polite, default)   9 SIGKILL (force)   1 SIGHUP (reload)
```
