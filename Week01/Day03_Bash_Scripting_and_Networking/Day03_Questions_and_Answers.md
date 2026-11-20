<h1 align="left">Day 03 – Bash Scripting & Networking – Questions & Answers</h1>

<p>
<img src="https://img.shields.io/badge/Week-01-1F3864?style=flat-square" alt="Week: 01">
<img src="https://img.shields.io/badge/Day-03-2E75B6?style=flat-square" alt="Day: 03">
<img src="https://img.shields.io/badge/Cloud-Local-FF9900?style=flat-square" alt="Cloud: Local">
<img src="https://img.shields.io/badge/Class-8%3A00%E2%80%9312%3A15%20EST-2E75B6?style=flat-square" alt="Class: 8:00–12:15 EST">
<img src="https://img.shields.io/badge/Lab-1%3A15%E2%80%935%3A00%20EST-2E7D32?style=flat-square" alt="Lab: 1:15–5:00 EST">
</p>

<p><a href="Day03_Bash_Scripting_and_Networking.md">Notes</a> · <a href="Day03_Commands_and_Setup.md">Commands & Setup</a> · <b>Questions & Answers</b> · <a href="../README.md">Week overview</a> · <a href="../../README.md">Course index</a></p>



Companion to the main notes: [Day03_Bash_Scripting_and_Networking.md](Day03_Bash_Scripting_and_Networking.md). All questions from today in one place: recap, interview Q&A,
live quiz and the answer key. Use it to revise and to prepare for client interviews.


<h2>1. Stand-up Recap Questions (8:00 AM)</h2>


Answer these from memory before class starts – they review the previous day.

1. What does `chmod 640 app.env` allow, and for whom?
2. A service shows `status=203/EXEC` in `systemctl status`. What do you check first?
3. Why must you use full paths inside a cron job?
4. What is the difference between `kill` and `kill -9`?
5. `df -h` says the disk is full, but `du` cannot find the big files. What could it be?


<h2>2. Interview-style Questions with Model Answers</h2>


<h3>1. "What does set -euo pipefail do, and why do you use it?"</h3>

<b>Model answer:</b> `-e` stops the script at the first failing command, `-u` makes using an unset variable an error, and `-o pipefail` makes a pipeline fail if any command in it fails, not only the last one. Together they stop a script from silently continuing after an error. This matters most in CI/CD, where a hidden failure can ship a broken build. I add a `trap ... ERR` to print the failing line and run ShellCheck in the pipeline.

<h3>2. "A user says the website is down. How do you troubleshoot?"</h3>

<b>Model answer:</b> I go layer by layer and stop at the first failure. DNS: does `dig` resolve the name to the expected IP? Network: do `nc -zv host 443` or `curl -v` get refused or time out? TLS: does `openssl s_client -servername` show a valid chain, name and dates? App: `curl -i` for the status code, then the app and load balancer logs. A 502 or 503 points me at the load balancer's target health and the backends.

<h3>3. "You have a 10.0.0.0/16 VPC. How many /24 subnets can you create, and how many usable IPs does each have on AWS?"</h3>

<b>Model answer:</b> 256 subnets of `/24`. Each has 256 addresses, and AWS reserves 5 (network, VPC router, DNS, future use and the last address), so 251 usable per subnet. Azure also reserves 5.

<h3>4. "What is the difference between connection refused and connection timed out?"</h3>

<b>Model answer:</b> Refused means the host answered with a reset: the network path works, but nothing listens on that port, or a firewall actively rejects it. Timed out means no reply at all. The usual causes are a security group, NSG, NACL or firewall silently dropping packets, a wrong route, or a host that is down. Refused sends me to the app; timeout sends me to the network rules.

<h3>5. "How does TLS verify a server, and what are common certificate problems?"</h3>

<b>Model answer:</b> The client checks that the certificate chains to a trusted CA, that the hostname is in the Subject Alternative Names, and that today is within the validity dates. Common problems are an expired certificate, a missing intermediate, a wrong hostname, or missing SNI. I debug with `openssl s_client -connect host:443 -servername host -showcerts`, and I automate renewal with ACM or cert-manager.

<h3>6. "What happens when you type a URL into a browser?"</h3>

<b>Model answer:</b> The browser resolves the name through DNS (cache, resolver, root, TLD, authoritative). It opens a TCP connection to port 443, does the TLS handshake with SNI and verifies the certificate, then sends an HTTP request. Usually a load balancer receives it and forwards it to a healthy backend, which returns a response that the browser renders, opening more connections for images and scripts.

<h3>7. "How do you securely access a server in a private subnet?"</h3>

<b>Model answer:</b> My first choice is AWS Systems Manager Session Manager or Azure Bastion: no public IP and no open port 22, and every session is audited. Otherwise a bastion host with key-only SSH, restricted to my IP, using `ssh -J bastion private-host`. Never password authentication and never port 22 open to `0.0.0.0/0`.

<h3>8. "How would you check what is listening on a Linux server?"</h3>

<b>Model answer:</b> `ss -ltnp` (or `netstat -tulpn` on older systems) shows listening ports, the bind address and the process. I check whether it binds to `127.0.0.1` (local only) or `0.0.0.0` (all interfaces), and then confirm from another machine with `nc -zv` that the firewall allows it.


<h2>3. Live Quiz (10 questions)</h2>


Try every question first, then check the answer key in section 5.

<b>Q1.</b> How many total addresses are in a `/26`?
- A) 26
- B) 32
- C) 64
- D) 128

<b>Q2.</b> What does this print? `x=; echo "[${x:-default}]"`
- A) `[]`
- B) `[default]`
- C) `[x]`
- D) an error

<b>Q3.</b> True or false: without `pipefail`, `false | true` returns exit code 0.
- A) True
- B) False

<b>Q4.</b> Which DNS record type maps a name to an IPv4 address?
- A) `CNAME`
- B) `MX`
- C) `A`
- D) `TXT`

<b>Q5.</b> `curl` exits with code 7. What does it mean?
- A) Timeout
- B) Could not connect (refused or no route)
- C) TLS certificate problem
- D) HTTP 404

<b>Q6.</b> Which tool shows the certificate chain and expiry date of a remote HTTPS server?
- A) `dig`
- B) `ss`
- C) `openssl s_client`
- D) `traceroute`

<b>Q7.</b> A load balancer returns <b>502</b>. What is the most likely cause?
- A) The DNS record is wrong
- B) The client sent bad JSON
- C) The load balancer cannot get a valid response from the backend
- D) The client is being rate limited

<b>Q8.</b> Which of these is NOT an RFC 1918 private range?
- A) `10.0.0.0/8`
- B) `172.16.0.0/12`
- C) `192.168.0.0/16`
- D) `100.64.0.0/10`

<b>Q9.</b> What happens under `set -e`? `count=0; ((count++)); echo done`
- A) prints `done`
- B) nothing – the script exits
- C) prints `1`
- D) prints `0`

<b>Q10.</b> True or false: `ss -ltn` showing `127.0.0.1:8080` means other machines can reach the service on port 8080.
- A) True
- B) False


<h2>5. Answer Key</h2>


1. C – 2^(32 − 26) = 64 addresses.
2. B – `:-` gives the default when the variable is unset <b>or empty</b>.
3. A (True) – without `pipefail`, a pipeline's exit code is the last command's (`true` gives 0).
4. C – `A` record (`AAAA` is IPv6).
5. B – exit 7 = failed to connect; 28 is timeout, 60 is a certificate problem.
6. C – `openssl s_client -connect host:443 -servername host`.
7. C – 502 Bad Gateway: the backend failed or returned an invalid response.
8. D – `100.64.0.0/10` is carrier-grade NAT space (RFC 6598), not RFC 1918.
9. B – `((count++))` evaluates to 0 (false), so it returns exit code 1 and `set -e` exits.
10. B (False) – it listens only on loopback; it must bind to `0.0.0.0` or the host IP.
