<h1 align="left">Day 03 – Bash Scripting & Networking – Commands & Setup Steps</h1>

<p>
<img src="https://img.shields.io/badge/Week-01-1F3864?style=flat-square" alt="Week: 01">
<img src="https://img.shields.io/badge/Day-03-2E75B6?style=flat-square" alt="Day: 03">
<img src="https://img.shields.io/badge/Cloud-Local-FF9900?style=flat-square" alt="Cloud: Local">
<img src="https://img.shields.io/badge/Class-8%3A00%E2%80%9312%3A15%20EST-2E75B6?style=flat-square" alt="Class: 8:00–12:15 EST">
<img src="https://img.shields.io/badge/Lab-1%3A15%E2%80%935%3A00%20EST-2E7D32?style=flat-square" alt="Lab: 1:15–5:00 EST">
</p>

<p><a href="Day03_Bash_Scripting_and_Networking.md">Notes</a> · <b>Commands & Setup</b> · <a href="Day03_Questions_and_Answers.md">Questions & Answers</a> · <a href="../README.md">Week overview</a> · <a href="../../README.md">Course index</a></p>



Companion to the main notes: [Day03_Bash_Scripting_and_Networking.md](Day03_Bash_Scripting_and_Networking.md). Every command used today, in the order you meet it in class and lab,
with the command breakdown tables that explain each part. Run commands in your own terminal and replace
`<your-name>` with your resource name from the Batch Roster.


<h2>1. Installation & Setup Steps</h2>


Run these before or at the start of the lab (installs, logins, cluster start-up, tool configuration).

<b>From:</b> Prerequisites

```bash
sudo apt-get update
sudo apt-get install -y curl dnsutils traceroute iproute2 iputils-tracepath openssl netcat-openbsd jq shellcheck
# Amazon Linux 2023 / RHEL: sudo dnf install -y curl bind-utils traceroute iproute openssl nmap-ncat jq
```


<h2>2. Commands by Session</h2>


<h3>8:15 – 9:45 AM · Concept Session 1</h3>


<h4>2. Bash Scripts – Shebang and Running a Script</h4>


```bash
mkdir -p ~/day03 && cd ~/day03
cat > hello.sh <<'EOF'
#!/usr/bin/env bash
# hello.sh - the smallest useful script
set -euo pipefail                  # strict mode - explained in section 7

name="${1:-world}"                 # first argument, default "world"
echo "Hello, ${name}! Today is $(date +%A)."
EOF
chmod +x hello.sh
./hello.sh
./hello.sh Cindia
bash hello.sh Raymundo             # run with bash directly (no execute bit needed)
```

Expected output:

```text
Hello, world! Today is Wednesday.
Hello, Cindia! Today is Wednesday.
Hello, Raymundo! Today is Wednesday.
```

<b>Command breakdown:</b> `cat > hello.sh <<'EOF' ... EOF`

| Part | What it means |
|---|---|
| `cat > hello.sh` | write what `cat` receives into `hello.sh` (overwrite if it exists) |
| `<<'EOF'` | here-document: the lines until `EOF` are the input |
| `'EOF'` (quoted) | do NOT expand `$` variables while writing – the script keeps `${1:-world}` as text |
| `EOF` (last line) | end marker; it must be alone on its line with no spaces before it |
| `chmod +x hello.sh` | add the execute bit so `./hello.sh` works; without it you get "Permission denied" |
| `./hello.sh` | run the file in the current folder; `hello.sh` alone fails because `.` is not on `PATH` |

<h4>3. Variables, Quoting and Parameter Expansion</h4>


```bash
env=dev
app="PeoplePulse API"                  # quotes needed because of the space
echo "Deploying ${app} to ${env}"
echo 'Deploying ${app}'                # single quotes - printed literally
files=("q3 report.csv" "employees.csv")
for f in "${files[@]}"; do echo "file: $f"; done
for f in ${files[@]}; do echo "unquoted: $f"; done   # WRONG - splits on spaces
unset AWS_REGION
echo "region=${AWS_REGION:-us-east-1}"
: "${AWS_REGION:?AWS_REGION must be set}"
```

Expected output:

```text
Deploying PeoplePulse API to dev
Deploying ${app}
file: q3 report.csv
file: employees.csv
unquoted: q3
unquoted: report.csv
unquoted: employees.csv
region=us-east-1
bash: AWS_REGION: AWS_REGION must be set
```

<b>Command breakdown:</b> `: "${AWS_REGION:?AWS_REGION must be set}"`

| Part | What it means |
|---|---|
| `:` | the "do nothing" command – it exists only so Bash evaluates the argument |
| `"${AWS_REGION ... }"` | read the variable, inside double quotes |
| `:?` | if the variable is unset <b>or empty</b>, print the message and stop the script (exit 1) |
| `AWS_REGION must be set` | the error message the user sees – make it say how to fix the problem |
| `${AWS_REGION:-us-east-1}` (other example) | `:-` gives a default instead of stopping |

<h4>4. Conditions, Tests, Loops and case</h4>


```bash
# if / elif / else
if [[ -f /etc/os-release ]]; then
  . /etc/os-release                     # "source" the file: load its variables
  echo "OS: ${PRETTY_NAME}"
elif [[ "$(uname)" == "Darwin" ]]; then
  echo "macOS"
else
  echo "unknown OS" >&2
fi

# for loop over a list
for env in dev staging prod; do
  echo "would deploy to ${env}"
done

# C-style loop
for (( i = 1; i <= 3; i++ )); do echo "attempt $i"; done

# while loop reading a file line by line (safe for spaces)
printf '# servers\nweb-01\n\napi-02\n' > servers.txt
while IFS= read -r host; do
  [[ -z "$host" || "$host" == \#* ]] && continue    # skip blanks and comments
  echo "checking $host"
done < servers.txt

# case - a small sub-command menu
action="${1:-status}"
case "$action" in
  start)  echo "starting" ;;
  stop)   echo "stopping" ;;
  status) echo "all good" ;;
  *)      echo "usage: $0 {start|stop|status}" >&2; exit 2 ;;
esac
```

Expected output:

```text
OS: Ubuntu 24.04.1 LTS
would deploy to dev
would deploy to staging
would deploy to prod
attempt 1
attempt 2
attempt 3
checking web-01
checking api-02
all good
```

<b>Command breakdown:</b> `while IFS= read -r host; do ... done < servers.txt`

| Part | What it means |
|---|---|
| `while ... do ... done` | repeat the body as long as the condition succeeds |
| `IFS=` | empty field separator – keeps leading and trailing spaces in the line |
| `read -r host` | read one line into `host`; `-r` keeps backslashes as normal characters |
| `< servers.txt` | feed the file into the loop; `read` fails at end of file, which ends the loop |
| `[[ ... == \#* ]]` | line starts with `#` (the backslash stops `#` being read as a comment) |
| `&& continue` | if the test is true, skip to the next line |

<h4>5. Functions, Arguments and getopts</h4>


```bash
cat > greet.sh <<'EOF'
#!/usr/bin/env bash
set -euo pipefail

log()   { printf '%s [%s] %s\n' "$(date -u +%FT%TZ)" "$1" "$2" >&2; }
die()   { log ERROR "$1"; exit "${2:-1}"; }
usage() { echo "Usage: $0 [-t team] name..." >&2; exit 2; }

greet() {
  local person="$1" team="$2"          # local: only visible inside greet()
  echo "Welcome ${person} to the ${team} team"
}

team="devops"
while getopts ":t:h" opt; do
  case "$opt" in
    t) team="$OPTARG" ;;
    h|*) usage ;;
  esac
done
shift $((OPTIND - 1))                  # remove the options; names remain in "$@"
[[ $# -gt 0 ]] || usage

for person in "$@"; do
  greet "$person" "$team"
done
log INFO "greeted $# people"
EOF
chmod +x greet.sh
./greet.sh Haiman Jobanjit
./greet.sh -t qa Carla
./greet.sh; echo "exit=$?"
```

Expected output:

```text
Welcome Haiman to the devops team
Welcome Jobanjit to the devops team
2026-11-11T13:20:05Z [INFO] greeted 2 people
Welcome Carla to the qa team
2026-11-11T13:20:06Z [INFO] greeted 1 people
Usage: ./greet.sh [-t team] name...
exit=2
```

<b>Command breakdown:</b> `while getopts ":t:h" opt; do ... done; shift $((OPTIND - 1))`

| Part | What it means |
|---|---|
| `getopts` | built-in Bash option reader; called once per option in a loop |
| `":t:h"` | leading `:` = silent mode (you handle errors); `t:` = `-t` needs a value; `h` = `-h` has no value |
| `opt` | variable that receives the option letter (`t`, `h`, or `?` for unknown) |
| `$OPTARG` | the value given after `-t` |
| `$OPTIND` | index of the next argument still to process |
| `shift $((OPTIND - 1))` | throw away the options so only the names remain in `"$@"` |

<h4>6. Exit Codes, stdout/stderr and Redirection</h4>


```bash
ls /etc/hostname /nope; echo "exit=$?"
ls /etc/hostname /nope > out.txt 2> err.txt; cat out.txt err.txt
ls /etc/hostname /nope > all.txt 2>&1; wc -l all.txt
nosuchcommand; echo "exit=$?"
grep -q root /etc/passwd && echo "root user exists"
grep -q nobody-here /etc/passwd || echo "not found - handled"
```

Expected output:

```text
ls: cannot access '/nope': No such file or directory
/etc/hostname
exit=2
/etc/hostname
ls: cannot access '/nope': No such file or directory
2 all.txt
nosuchcommand: command not found
exit=127
root user exists
not found - handled
```

<b>Command breakdown:</b> `ls /etc/hostname /nope > all.txt 2>&1`

| Part | What it means |
|---|---|
| `ls /etc/hostname /nope` | one file exists (stdout), one does not (stderr) |
| `> all.txt` | send stream 1 (stdout) to `all.txt` |
| `2>&1` | send stream 2 (stderr) to wherever stream 1 points <b>now</b> |
| order matters | `2>&1 > all.txt` sends errors to the screen, because stdout was still the screen when `2>&1` ran |

<h4>7. Strict Mode: set -euo pipefail, trap and ShellCheck</h4>


```bash
cat > strict-demo.sh <<'EOF'
#!/usr/bin/env bash
set -euo pipefail
trap 'echo "ERROR on line $LINENO (exit $?)" >&2' ERR
tmpfile=$(mktemp)
trap 'rm -f "$tmpfile"' EXIT                    # runs on every exit, success or failure

curl -fsS -m 5 https://api.github.com/zen > "$tmpfile"
echo "GitHub says: $(cat "$tmpfile")"
curl -fsS -m 5 https://api.github.com/this-does-not-exist > "$tmpfile"
echo "never printed"
EOF
chmod +x strict-demo.sh
./strict-demo.sh; echo "exit=$?"
false | true; echo "no pipefail: $?"
set -o pipefail; false | true; echo "with pipefail: $?"; set +o pipefail
```

Expected output:

```text
GitHub says: Design for failure.
curl: (22) The requested URL returned error: 404
ERROR on line 9 (exit 22)
exit=22
no pipefail: 0
with pipefail: 1
```

<b>Command breakdown:</b> `trap 'echo "ERROR on line $LINENO (exit $?)" >&2' ERR`

| Part | What it means |
|---|---|
| `trap '...' ERR` | run the quoted command whenever a command fails (with `set -e`, just before exit) |
| `$LINENO` | line number where the failure happened |
| `$?` | the failing command's exit code |
| `>&2` | print to stderr so it shows in CI logs and is not captured as data |
| `trap '...' EXIT` (second trap) | runs on every exit – the right place to delete temp files |
| single quotes | the variables are expanded when the trap fires, not when it is defined |

<b>Command breakdown:</b> `curl -fsS -m 5 URL`

| Part | What it means |
|---|---|
| `-f` | fail (exit 22) on HTTP 400 or higher; without it, a 404 page counts as success |
| `-s` | silent – no progress bar |
| `-S` | but still <b>show</b> errors when `-s` is on |
| `-m 5` | give up after 5 seconds in total; without it a hung server blocks your script for minutes |

<h4>8. OSI and TCP/IP Models; TCP vs UDP</h4>


```bash
nc -zv -w 3 github.com 443          # TCP port open?
nc -zv -w 3 127.0.0.1 9999          # nothing listening locally
```

Expected output:

```text
Connection to github.com (140.82.112.3) 443 port [tcp/https] succeeded!
nc: connect to 127.0.0.1 port 9999 (tcp) failed: Connection refused
```

<b>Command breakdown:</b> `nc -zv -w 3 github.com 443`

| Part | What it means |
|---|---|
| `nc` | netcat – opens raw TCP/UDP connections |
| `-z` | zero-I/O mode: only check the port opens, send no data |
| `-v` | verbose: print "succeeded" or the error |
| `-w 3` | wait at most 3 seconds; otherwise a dropped packet can hang a long time |
| `github.com 443` | host and port to test |
| `-u` (other use) | test UDP instead of TCP (results are less reliable, UDP has no handshake) |

<h4>9. IP Addresses, CIDR and Subnetting</h4>


```bash
python3 - <<'EOF'
import ipaddress
vpc = ipaddress.ip_network("10.20.0.0/16")
print("total addresses:", vpc.num_addresses)
subs = list(vpc.subnets(new_prefix=24))
print("number of /24s:", len(subs))
print("first three:", [str(s) for s in subs[:3]])
s = ipaddress.ip_network("10.20.10.0/24")
hosts = list(s.hosts())
print("10.20.10.0/24 AWS usable range:", hosts[3], "-", hosts[-1], "=", s.num_addresses - 5, "IPs")
EOF
```

Expected output:

```text
total addresses: 65536
number of /24s: 256
first three: ['10.20.0.0/24', '10.20.1.0/24', '10.20.2.0/24']
10.20.10.0/24 AWS usable range: 10.20.10.4 - 10.20.10.254 = 251 IPs
```

<b>Command breakdown:</b> `python3 - <<'EOF' ... EOF` with `ipaddress`

| Part | What it means |
|---|---|
| `python3 -` | run Python code read from stdin (the `-`) |
| `<<'EOF'` | the here-document supplies that code |
| `ipaddress.ip_network("10.20.0.0/16")` | build a network object from a CIDR string |
| `.subnets(new_prefix=24)` | split it into every possible `/24` |
| `.hosts()` | all addresses except network and broadcast; AWS also reserves `.1`–`.3`, so we start at index 3 (`.4`) |
| `num_addresses - 5` | usable IPs in AWS or Azure |

<h3>10:00 – 11:15 AM · Concept Session 2 + Live Demo</h3>


<h4>10. DNS – the Internet's Phone Book</h4>


```bash
dig github.com +short                     # just the IP(s)
dig github.com A +noall +answer           # the answer line with TTL
dig @1.1.1.1 github.com +short            # ask a specific resolver
dig www.github.com CNAME +short           # alias target
dig nope.peoplepulse.invalid | grep status
getent hosts github.com                   # what the OS resolver returns (includes /etc/hosts)
resolvectl status | grep -m1 "DNS Servers"
```

Expected output:

```text
140.82.112.3
github.com.             60      IN      A       140.82.112.3
140.82.112.3
github.com.
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 41872
140.82.112.3    github.com
Current DNS Server: 172.31.0.2
```

<b>Command breakdown:</b> `dig github.com A +noall +answer`

| Part | What it means |
|---|---|
| `dig` | DNS lookup tool – asks DNS servers directly, ignores `/etc/hosts` |
| `github.com` | the name to look up |
| `A` | record type (default is `A`); try `AAAA`, `CNAME`, `MX`, `TXT`, `NS` |
| `+noall +answer` | hide everything except the answer section |
| `60` in the output | TTL: seconds this answer may be cached |
| `@1.1.1.1` (other example) | ask this resolver instead of the system default |
| `+trace` (other use) | follow the delegation from the root yourself – shows where it breaks |
| `+short` | print only the values |

<h4>11. HTTP, HTTPS and curl</h4>


```bash
curl -i https://api.github.com/zen                     # -i: show response headers too
curl -sS -o /dev/null -w '%{http_code} %{time_total}s\n' https://github.com
curl -sS -X POST -H 'Content-Type: application/json' \
  -d '{"name":"test"}' https://httpbin.org/post | jq .json
curl -sSL -o /dev/null -w '%{http_code} %{url_effective}\n' http://github.com
```

Expected output:

```text
HTTP/2 200
content-type: text/plain;charset=utf-8
...
Keep it logically awesome.
200 0.183214s
{
  "name": "test"
}
200 https://github.com/
```

<b>Command breakdown:</b> `curl -sS -o /dev/null -w '%{http_code} %{time_total}s\n' https://github.com`

| Part | What it means |
|---|---|
| `-s` / `-S` | silent, but still show errors |
| `-o /dev/null` | throw the body away – we only want numbers |
| `-w '...'` | "write out" these values after the request finishes |
| `%{http_code}` | the HTTP status code; `000` if no HTTP answer came back at all |
| `%{time_total}` | total seconds for the request (DNS + connect + TLS + response) |
| `-X POST -H ... -d ...` (other example) | method, a header, and a request body |
| `-L` (other example) | follow redirects; `%{url_effective}` shows where you ended up |
| `--resolve host:443:IP` (useful extra) | test a new server's IP <b>before</b> changing DNS |

<h4>12. TLS Certificates</h4>


```bash
openssl s_client -connect github.com:443 -servername github.com </dev/null 2>/dev/null \
  | openssl x509 -noout -subject -issuer -dates -ext subjectAltName
```

Expected output:

```text
subject=CN = github.com
issuer=C = GB, ST = Greater Manchester, L = Salford, O = Sectigo Limited, CN = Sectigo ECC Domain Validation Secure Server CA
notBefore=Feb  5 00:00:00 2026 GMT
notAfter=Feb  5 23:59:59 2027 GMT
X509v3 Subject Alternative Name:
    DNS:github.com, DNS:www.github.com
```

<b>Command breakdown:</b> `openssl s_client -connect github.com:443 -servername github.com </dev/null 2>/dev/null | openssl x509 -noout -subject -issuer -dates -ext subjectAltName`

| Part | What it means |
|---|---|
| `openssl s_client` | act as a TLS client and print the handshake details |
| `-connect github.com:443` | host and port to connect to |
| `-servername github.com` | send SNI; without it a shared load balancer may return its <b>default</b> certificate |
| `</dev/null` | send no input, so the connection closes at once instead of waiting |
| `2>/dev/null` | hide handshake noise on stderr |
| `\| openssl x509` | read the certificate that `s_client` printed |
| `-noout` | do not print the encoded certificate itself |
| `-subject -issuer -dates` | who it is for, who signed it, valid from/to |
| `-ext subjectAltName` | the list of names (SANs) the certificate covers |

<h4>13. SSH Keys</h4>


```bash
ssh-keygen -t ed25519 -C "<your-name>@devops-8wk" -f ~/.ssh/id_ed25519   # choose a passphrase
chmod 700 ~/.ssh && chmod 600 ~/.ssh/id_ed25519
ls -l ~/.ssh/id_ed25519*
ssh-copy-id -i ~/.ssh/id_ed25519.pub ubuntu@<server-ip>    # install the public key on a server
cat >> ~/.ssh/config <<'EOF'
Host lab
  HostName <server-ip>
  User ubuntu
  IdentityFile ~/.ssh/id_ed25519
EOF
ssh lab                                                    # uses the settings above
ssh -L 5432:db.internal:5432 lab                           # tunnel to a private database
```

Expected output:

```text
-rw------- 1 jobanjit jobanjit 411 Nov 11 10:33 /home/jobanjit/.ssh/id_ed25519
-rw-r--r-- 1 jobanjit jobanjit 101 Nov 11 10:33 /home/jobanjit/.ssh/id_ed25519.pub
```

<b>Command breakdown:</b> `ssh-keygen -t ed25519 -C "<your-name>@devops-8wk" -f ~/.ssh/id_ed25519`

| Part | What it means |
|---|---|
| `ssh-keygen` | create a new key pair |
| `-t ed25519` | key type: modern, short and fast; use `-t rsa -b 4096` only if an old system needs RSA |
| `-C "..."` | comment saved in the public key – helps you see whose key it is on a server |
| `-f ~/.ssh/id_ed25519` | file name; the public key is the same name plus `.pub` |
| passphrase prompt | encrypts the private key on disk; use `ssh-agent` so you type it once per session |

<b>Command breakdown:</b> `ssh -L 5432:db.internal:5432 lab`

| Part | What it means |
|---|---|
| `-L` | local port forward (a tunnel) |
| `5432` (first) | port opened on <b>your laptop</b> |
| `db.internal:5432` | where the server `lab` sends the traffic – a host only it can reach |
| `lab` | the jump server from `~/.ssh/config` |
| result | `psql -h 127.0.0.1 -p 5432` on your laptop reaches the private database |

<h4>14. Ports, ss and netstat</h4>


```bash
python3 -m http.server 8080 --bind 127.0.0.1 > /dev/null 2>&1 &
sleep 1
ss -ltnp
ss -tn state established '( dport = :443 )' | head -3
sudo netstat -tulpn 2>/dev/null | head -3        # older tool (package net-tools)
kill %1
```

Expected output:

```text
State   Recv-Q  Send-Q   Local Address:Port    Peer Address:Port  Process
LISTEN  0       5            127.0.0.1:8080         0.0.0.0:*      users:(("python3",pid=6120,fd=3))
LISTEN  0       4096           0.0.0.0:22           0.0.0.0:*      users:(("sshd",pid=812,fd=3))
LISTEN  0       4096     127.0.0.53%lo:53           0.0.0.0:*
Recv-Q  Send-Q   Local Address:Port      Peer Address:Port
0       0        172.24.10.5:51122       140.82.112.3:443
```

<b>Command breakdown:</b> `ss -ltnp`

| Part | What it means |
|---|---|
| `ss` | socket statistics – modern replacement for `netstat` |
| `-l` | only <b>listening</b> sockets (servers waiting for connections) |
| `-t` | TCP only (`-u` for UDP) |
| `-n` | show numbers (`8080`), not names (`http-alt`); also faster |
| `-p` | show the process; needs `sudo` for processes owned by other users |
| `Local Address:Port` column | `127.0.0.1` = local only; `0.0.0.0` = all interfaces |
| `state established '( dport = :443 )'` (second example) | filter: open connections to remote port 443 |

<h4>15. Following the Path: ping, traceroute, tracepath and nc</h4>


```bash
ip -br addr                     # my interfaces and IPs
ip route | head -2              # default gateway
ping -c 3 1.1.1.1
tracepath -n 1.1.1.1 | head -5
```

Expected output:

```text
lo               UNKNOWN        127.0.0.1/8 ::1/128
eth0             UP             172.24.10.5/20
default via 172.24.0.1 dev eth0 proto kernel
172.24.0.0/20 dev eth0 proto kernel scope link src 172.24.10.5
64 bytes from 1.1.1.1: icmp_seq=1 ttl=56 time=11.8 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=56 time=12.1 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=56 time=11.9 ms
 1?: [LOCALHOST]                      pmtu 1500
 1:  172.24.0.1                                            0.412ms
 2:  192.168.1.1                                           2.871ms
 3:  100.64.0.1                                            9.405ms
 4:  no reply
```

<b>Command breakdown:</b> `sudo traceroute -T -p 443 -n github.com`

| Part | What it means |
|---|---|
| `traceroute` | list each hop on the way to the host |
| `-T` | use TCP SYN packets instead of UDP – they pass firewalls that allow HTTPS |
| `-p 443` | destination port for the TCP probes |
| `-n` | show IPs only, do not look up names (faster, no DNS noise) |
| `sudo` | TCP mode needs raw sockets, which need root |
| `* * *` in output | no reply from that hop – keep reading; only worry if <b>all</b> later hops are `*` |

<h4>Live Demo: "Is It DNS, Network, TLS or the App?"</h4>


```bash
mkdir -p ~/day03/www && echo '{"status":"ok"}' > ~/day03/www/health
python3 -m http.server 8080 --bind 127.0.0.1 --directory ~/day03/www > ~/day03/server.log 2>&1 &
echo $! > ~/day03/server.pid
sleep 1
ss -ltnp | grep 8080
```

Expected output:

```text
LISTEN 0      5          127.0.0.1:8080      0.0.0.0:*    users:(("python3",pid=6120,fd=3))
```

<b>Command breakdown:</b> `python3 -m http.server 8080 --bind 127.0.0.1 --directory ~/day03/www > ~/day03/server.log 2>&1 &`

| Part | What it means |
|---|---|
| `python3 -m http.server 8080` | Python's built-in web server on port 8080 |
| `--bind 127.0.0.1` | listen only on loopback (this machine) |
| `--directory ~/day03/www` | serve files from this folder, so `/health` returns our JSON file |
| `> ~/day03/server.log 2>&1` | send both output streams to a log file |
| `&` | run in the background and give the prompt back |
| `echo $! > server.pid` | `$!` is the PID of the last background job; we save it to stop the server later |

```bash
curl -sS -o /dev/null -w 'HTTP %{http_code} in %{time_total}s\n' http://127.0.0.1:8080/health
curl -sS -o /dev/null -w 'HTTP %{http_code}\n' http://127.0.0.1:8080/nope
```

Expected output:

```text
HTTP 200 in 0.002134s
HTTP 404
```

```bash
curl -sS -m 3 http://127.0.0.1:9999/health; echo "exit=$?"
curl -sS -m 3 http://10.255.255.1/health; echo "exit=$?"
```

Expected output:

```text
curl: (7) Failed to connect to 127.0.0.1 port 9999 after 0 ms: Couldn't connect to server
exit=7
curl: (28) Connection timed out after 3002 milliseconds
exit=28
```

```bash
dig +short github.com
dig doesnotexist.peoplepulse.invalid | grep status
curl -sS https://doesnotexist.peoplepulse.invalid; echo "exit=$?"
```

Expected output:

```text
140.82.112.3
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 41872
curl: (6) Could not resolve host: doesnotexist.peoplepulse.invalid
exit=6
```

```bash
openssl s_client -connect github.com:443 -servername github.com </dev/null 2>/dev/null \
  | openssl x509 -noout -subject -enddate
curl -sS https://expired.badssl.com/ > /dev/null; echo "exit=$?"
```

Expected output:

```text
subject=CN = github.com
notAfter=Feb  5 23:59:59 2027 GMT
curl: (60) SSL certificate problem: certificate has expired
exit=60
```

```bash
for url in http://127.0.0.1:8080/health http://127.0.0.1:9999/health; do
  if curl -fsS -m 3 -o /dev/null "$url"; then echo "OK   $url"; else echo "FAIL $url"; fi
done
kill "$(cat ~/day03/server.pid)"
```

Expected output:

```text
OK   http://127.0.0.1:8080/health
curl: (7) Failed to connect to 127.0.0.1 port 9999 after 0 ms: Couldn't connect to server
FAIL http://127.0.0.1:9999/health
```

<b>Command breakdown:</b> `if curl -fsS -m 3 -o /dev/null "$url"; then ... else ... fi`

| Part | What it means |
|---|---|
| `if <command>; then` | `if` runs the command and looks only at its exit code (0 = true) |
| `curl -f` | turn HTTP 4xx/5xx into a non-zero exit code, so a 404 counts as failure |
| `-m 3` | 3-second limit per URL |
| `"$url"` | quoted, so a URL with `&` or spaces stays one argument |
| `set -e` note | a failing command inside an `if` does <b>not</b> stop a strict-mode script – that is why this pattern is safe |

<h3>1:15 – 2:45 PM · Lab Part 1 – Task Assignment & Guided Build</h3>


<h4>Task 1 – Set Up Test Targets</h4>


```bash
mkdir -p ~/devops-8wk/devops-8wk-<your-name>/day03 && cd ~/devops-8wk/devops-8wk-<your-name>/day03
mkdir -p www && echo '{"status":"ok","app":"peoplepulse-stub"}' > www/health
python3 -m http.server 8080 --bind 127.0.0.1 --directory www > server.log 2>&1 &
echo $! > server.pid

cat > endpoints.txt <<'EOF'
# One URL per line. Lines starting with # are ignored.
http://127.0.0.1:8080/health
https://github.com
http://127.0.0.1:9999/health
EOF
curl -s 127.0.0.1:8080/health; echo
grep -vc '^#' endpoints.txt
```

Expected output:

```text
{"status":"ok","app":"peoplepulse-stub"}
3
```

<h4>Task 2 – Write healthcheck.sh</h4>


```bash
cat > healthcheck.sh <<'EOF'
#!/usr/bin/env bash
# healthcheck.sh - curl a list of HTTP(S) endpoints and alert on failure.
# Usage:      ./healthcheck.sh [-f endpoints.txt] [-t timeout_seconds] [url ...]
# Exit codes: 0 = all healthy, 1 = at least one endpoint failed, 2 = usage error.
set -euo pipefail

TIMEOUT="${TIMEOUT:-5}"                        # per-request timeout in seconds
RETRIES="${RETRIES:-2}"                        # extra attempts after the first
LOG_FILE="${LOG_FILE:-./healthcheck.log}"
SLACK_WEBHOOK_URL="${SLACK_WEBHOOK_URL:-}"     # optional - never hard-code it
ENDPOINTS_FILE=""

log() {
  printf '%s %s\n' "$(date -u +%Y-%m-%dT%H:%M:%SZ)" "$*" | tee -a "$LOG_FILE" >&2
}

usage() {
  echo "Usage: $0 [-f endpoints_file] [-t timeout_seconds] [url ...]" >&2
  exit 2
}

alert() {
  local message="$1" payload
  log "ALERT ${message}"
  if [[ -n "$SLACK_WEBHOOK_URL" ]]; then
    payload=$(jq -n --arg text "$message" '{text: $text}')   # safe JSON, even with quotes
    curl -sS -m 5 -X POST -H 'Content-Type: application/json' \
      --data "$payload" "$SLACK_WEBHOOK_URL" > /dev/null \
      || log "WARN could not send Slack alert"
  elif command -v logger > /dev/null; then
    logger -t healthcheck -p user.err "$message"            # goes to syslog / journal
  fi
}

check_url() {
  local url="$1" attempt code="000"
  for (( attempt = 1; attempt <= RETRIES + 1; attempt++ )); do
    # -w prints the status code; on connection errors curl prints 000 and exits non-zero
    code=$(curl -s -o /dev/null -w '%{http_code}' -m "$TIMEOUT" "$url" || true)
    if [[ "$code" =~ ^[23][0-9]{2}$ ]]; then
      log "OK   ${url} (HTTP ${code}, attempt ${attempt})"
      return 0
    fi
    if (( attempt <= RETRIES )); then sleep 1; fi
  done
  alert "FAIL ${url} (HTTP ${code}) after $((RETRIES + 1)) attempts on $(hostname)"
  return 1
}

main() {
  while getopts ":f:t:h" opt; do
    case "$opt" in
      f) ENDPOINTS_FILE="$OPTARG" ;;
      t) TIMEOUT="$OPTARG" ;;
      *) usage ;;
    esac
  done
  shift $((OPTIND - 1))

  local -a urls=("$@")
  local line
  if [[ -n "$ENDPOINTS_FILE" ]]; then
    [[ -r "$ENDPOINTS_FILE" ]] || { echo "Cannot read ${ENDPOINTS_FILE}" >&2; exit 2; }
    while IFS= read -r line; do
      [[ -z "$line" || "$line" == \#* ]] && continue
      urls+=("$line")
    done < "$ENDPOINTS_FILE"
  fi
  [[ ${#urls[@]} -gt 0 ]] || usage

  local failures=0 url
  for url in "${urls[@]}"; do
    check_url "$url" || failures=$((failures + 1))
  done

  log "SUMMARY total=${#urls[@]} failed=${failures}"
  if [[ "$failures" -gt 0 ]]; then
    exit 1
  fi
}

main "$@"
EOF
chmod +x healthcheck.sh
shellcheck healthcheck.sh && echo "shellcheck clean"
```

Expected output:

```text
shellcheck clean
```

<b>Command breakdown:</b> `code=$(curl -s -o /dev/null -w '%{http_code}' -m "$TIMEOUT" "$url" || true)`

| Part | What it means |
|---|---|
| `code=$( ... )` | store what the command prints (only the status code) in `code` |
| `curl -s -o /dev/null` | quiet, throw the body away |
| `-w '%{http_code}'` | print the status; prints `000` when there is no HTTP answer (refused, timeout, DNS) |
| `-m "$TIMEOUT"` | give up after this many seconds |
| `\|\| true` | curl exits non-zero on connection errors; `\|\| true` stops `set -e` from killing the script so we can retry |
| no `-f` here | on purpose: we want the real status code (404, 503) in the log, not just "failed" |

<b>Command breakdown:</b> `[[ "$code" =~ ^[23][0-9]{2}$ ]]`

| Part | What it means |
|---|---|
| `=~` | match against a regular expression (do not quote the pattern) |
| `^` / `$` | start and end of the text – the whole code must match |
| `[23]` | first digit 2 or 3 (success or redirect) |
| `[0-9]{2}` | exactly two more digits |
| result | `200`, `204`, `301` pass; `000`, `404`, `503` fail |

<b>Command breakdown:</b> `payload=$(jq -n --arg text "$message" '{text: $text}')`

| Part | What it means |
|---|---|
| `jq -n` | start with no input – we build JSON from scratch |
| `--arg text "$message"` | pass the shell variable into jq as a string named `text` |
| `'{text: $text}'` | build `{"text": "..."}`; jq escapes quotes and newlines correctly |
| why | building JSON with `"{\"text\": \"$message\"}"` breaks when the message contains a quote |

```bash
./healthcheck.sh -f endpoints.txt; echo "exit=$?"
```

Expected output:

```text
2026-11-11T18:05:01Z OK   http://127.0.0.1:8080/health (HTTP 200, attempt 1)
2026-11-11T18:05:01Z OK   https://github.com (HTTP 200, attempt 1)
2026-11-11T18:05:03Z ALERT FAIL http://127.0.0.1:9999/health (HTTP 000) after 3 attempts on lab-laptop
2026-11-11T18:05:03Z SUMMARY total=3 failed=1
exit=1
```

```bash
./healthcheck.sh http://127.0.0.1:8080/health; echo "exit=$?"
./healthcheck.sh; echo "exit=$?"
journalctl -t healthcheck -n 3 --no-pager
TIMEOUT=2 RETRIES=0 ./healthcheck.sh -t 1 http://10.255.255.1/; echo "exit=$?"
```

Expected output:

```text
2026-11-11T18:06:10Z OK   http://127.0.0.1:8080/health (HTTP 200, attempt 1)
2026-11-11T18:06:10Z SUMMARY total=1 failed=0
exit=0
Usage: ./healthcheck.sh [-f endpoints_file] [-t timeout_seconds] [url ...]
exit=2
Nov 11 13:05:03 lab-laptop healthcheck[7012]: FAIL http://127.0.0.1:9999/health (HTTP 000) after 3 attempts on lab-laptop
2026-11-11T18:06:12Z ALERT FAIL http://10.255.255.1/ (HTTP 000) after 1 attempts on lab-laptop
2026-11-11T18:06:12Z SUMMARY total=1 failed=1
exit=1
```

<h4>Task 3 – Subnet a /16 into Public and Private /24s</h4>


```bash
cat > check_subnets.py <<'EOF'
"""Check that every subnet is inside the VPC and no two subnets overlap."""
import ipaddress
import itertools

vpc = ipaddress.ip_network("10.20.0.0/16")
subnets = {
    "public-a": "10.20.0.0/24", "public-b": "10.20.1.0/24", "public-c": "10.20.2.0/24",
    "app-a": "10.20.10.0/24", "app-b": "10.20.11.0/24", "app-c": "10.20.12.0/24",
    "db-a": "10.20.20.0/24", "db-b": "10.20.21.0/24", "db-c": "10.20.22.0/24",
}
nets = {name: ipaddress.ip_network(cidr) for name, cidr in subnets.items()}
for name, net in nets.items():
    assert net.subnet_of(vpc), f"{name} is outside the VPC"
    print(f"{name:9} {str(net):15} usable_on_aws={net.num_addresses - 5}")
for (a, na), (b, nb) in itertools.combinations(nets.items(), 2):
    assert not na.overlaps(nb), f"{a} overlaps {b}"
print("OK: all subnets inside", vpc, "and no overlaps")
EOF
python3 check_subnets.py
```

Expected output:

```text
public-a  10.20.0.0/24    usable_on_aws=251
public-b  10.20.1.0/24    usable_on_aws=251
public-c  10.20.2.0/24    usable_on_aws=251
app-a     10.20.10.0/24   usable_on_aws=251
app-b     10.20.11.0/24   usable_on_aws=251
app-c     10.20.12.0/24   usable_on_aws=251
db-a      10.20.20.0/24   usable_on_aws=251
db-b      10.20.21.0/24   usable_on_aws=251
db-c      10.20.22.0/24   usable_on_aws=251
OK: all subnets inside 10.20.0.0/16 and no overlaps
```

<b>Command breakdown:</b> the two checks in `check_subnets.py`

| Part | What it means |
|---|---|
| `net.subnet_of(vpc)` | True if the subnet sits completely inside the VPC range |
| `assert ..., "message"` | stop with an error and this message if the check is False |
| `itertools.combinations(nets.items(), 2)` | every pair of subnets, once (a-b, a-c, ... no repeats) |
| `na.overlaps(nb)` | True if the two ranges share even one address |
| `{name:9}` | f-string padding: print the name in a 9-character column |

<h4>Task 4 – Debug DNS and TLS</h4>


```bash
dig github.com A +noall +answer              # answer and TTL
dig @8.8.8.8 github.com +short
dig @1.1.1.1 github.com +short               # compare two resolvers
dig +trace github.com | tail -4              # who is authoritative?
dig nope.github.invalid | grep -o 'status: [A-Z]*'
```

Expected output:

```text
github.com.             60      IN      A       140.82.112.3
140.82.112.3
140.82.112.3
github.com.             60      IN      A       140.82.112.3
;; Received 55 bytes from 205.251.193.165#53(ns-421.awsdns-52.com) in 12 ms

status: NXDOMAIN
```

```bash
echo "127.0.0.1 api.peoplepulse.test" | sudo tee -a /etc/hosts
getent hosts api.peoplepulse.test           # OS resolver reads /etc/hosts
dig +short api.peoplepulse.test             # dig asks DNS directly - empty
curl -s http://api.peoplepulse.test:8080/health; echo
```

Expected output:

```text
127.0.0.1 api.peoplepulse.test
127.0.0.1       api.peoplepulse.test

{"status":"ok","app":"peoplepulse-stub"}
```

```bash
for host in expired.badssl.com wrong.host.badssl.com self-signed.badssl.com untrusted-root.badssl.com; do
  echo "== $host"
  openssl s_client -connect "$host:443" -servername "$host" </dev/null 2>/dev/null \
    | grep -E 'Verify return code'
  curl -sS -o /dev/null "https://$host/" 2>&1 | head -1
done
```

Expected output:

```text
== expired.badssl.com
Verify return code: 10 (certificate has expired)
curl: (60) SSL certificate problem: certificate has expired
== wrong.host.badssl.com
Verify return code: 0 (ok)
curl: (60) SSL: no alternative certificate subject name matches target host name 'wrong.host.badssl.com'
== self-signed.badssl.com
Verify return code: 18 (self-signed certificate)
curl: (60) SSL certificate problem: self-signed certificate
== untrusted-root.badssl.com
Verify return code: 19 (self-signed certificate in certificate chain)
curl: (60) SSL certificate problem: self-signed certificate in certificate chain
```

```bash
openssl s_client -connect wrong.host.badssl.com:443 -servername wrong.host.badssl.com \
  -verify_hostname wrong.host.badssl.com </dev/null 2>/dev/null | grep 'Verify return code'
openssl s_client -connect wrong.host.badssl.com:443 -servername wrong.host.badssl.com </dev/null 2>/dev/null \
  | openssl x509 -noout -ext subjectAltName
```

Expected output:

```text
Verify return code: 62 (hostname mismatch)
X509v3 Subject Alternative Name:
    DNS:*.badssl.com, DNS:badssl.com
```

```bash
end=$(openssl s_client -connect github.com:443 -servername github.com </dev/null 2>/dev/null \
      | openssl x509 -noout -enddate | cut -d= -f2)
echo "github.com expires in $(( ( $(date -d "$end" +%s) - $(date +%s) ) / 86400 )) days"
```

Expected output:

```text
github.com expires in 86 days
```

<b>Command breakdown:</b> `$(( ( $(date -d "$end" +%s) - $(date +%s) ) / 86400 ))`

| Part | What it means |
|---|---|
| `openssl x509 -noout -enddate` | prints `notAfter=Feb  5 23:59:59 2027 GMT` |
| `cut -d= -f2` | split on `=` and keep field 2 – the date text |
| `date -d "$end" +%s` | convert that date to seconds since 1970 (GNU date; on macOS use `gdate`) |
| `date +%s` | now, in seconds since 1970 |
| `/ 86400` | seconds per day – the result is whole days left |

<h3>2:45 – 4:00 PM · Lab Part 2 – Independent Challenge</h3>


<h4>Challenge 3 – Break/Fix: the Deploy Script That "Always Succeeds"</h4>


```bash
#!/bin/bash
# broken-deploy.sh - "deploys" by checking the target, then printing success
TARGET=$1
ENVIROMENT=${ENV}
curl -s http://$TARGET/health | grep -q ok
echo "Deploying to $ENVIRONMENT"
count=0
for f in $(ls *.txt); do
  ((count++))
done
echo "Checked $count files"
echo "SUCCESS"
```

<h3>4:00 – 4:45 PM · Lab Review with Trainer & Teardown</h3>


<h4>Teardown Commands</h4>


```bash
cd ~/devops-8wk/devops-8wk-<your-name>/day03
kill "$(cat server.pid)" 2>/dev/null; rm -f server.pid server.log
kill "$(cat ~/day03/server.pid)" 2>/dev/null; rm -rf ~/day03
crontab -l 2>/dev/null | grep -v healthcheck.sh | crontab -     # remove the 5-minute job
sudo sed -i '/api\.peoplepulse\.test/d' /etc/hosts              # remove the hosts override
ss -ltn | grep -E ':8080|:9999' || echo "no test servers listening"
crontab -l 2>/dev/null | grep -c healthcheck || true
```

Expected output:

```text
no test servers listening
0
```

<b>Command breakdown:</b> `crontab -l 2>/dev/null | grep -v healthcheck.sh | crontab -`

| Part | What it means |
|---|---|
| `crontab -l` | print your current crontab |
| `2>/dev/null` | hide "no crontab for user" if it is empty |
| `grep -v healthcheck.sh` | keep every line <b>except</b> the health-check job |
| `crontab -` | install the result from stdin as your new crontab |
| warning | if `grep -v` removes every line it exits 1, but the empty crontab is still installed – that is fine here |

<h4>Cost Check</h4>


```bash
aws ec2 describe-instances \
  --filters Name=tag:course,Values=devops-8wk Name=instance-state-name,Values=pending,running,stopped \
  --query 'Reservations[].Instances[].InstanceId' --output text     # expect empty
az group list --tag course=devops-8wk -o table                      # expect empty
```

<h3>4:45 – 5:00 PM · Commit & Daily Report</h3>


```bash
cd ~/devops-8wk/devops-8wk-<your-name>
git switch main && git pull
git switch -c day03-bash-networking
cat >> .gitignore <<'EOF'
*.log
*.pid
EOF
git add .gitignore day03/healthcheck.sh day03/endpoints.txt day03/subnet-plan.md \
        day03/check_subnets.py day03/dns-tls-notes.md day03/broken-deploy.sh
git status                      # make sure no .log, .pid or secrets are staged
git commit -m "feat(day03): healthcheck.sh with retries and alerts, subnet plan, DNS/TLS notes"
git push -u origin day03-bash-networking
```


<h2>3. Cheat Sheet</h2>


| Area | Command | Purpose |
|---|---|---|
| Bash | `set -euo pipefail` | strict mode – top of every script |
| Bash | `trap 'echo "line $LINENO failed" >&2' ERR` / `trap 'rm -f "$tmp"' EXIT` | report failures / always clean up |
| Bash | `"${var:-default}"`, `: "${var:?message}"` | default value / required variable |
| Bash | `for x in "${arr[@]}"; do ...; done` | loop over an array safely |
| Bash | `while IFS= read -r line; do ...; done < file` | read a file line by line |
| Bash | `count=$((count + 1))` | safe increment under `set -e` |
| Bash | `getopts ":f:t:h" opt` + `shift $((OPTIND - 1))` | parse options |
| Bash | `cmd > out 2>&1`, `cmd \| tee -a log`, `cmd1 && cmd2 \|\| cmd3` | redirection and chaining |
| Lint | `shellcheck script.sh` | find bugs before running |
| IP | `ip -br addr`, `ip route` | my IPs and default gateway |
| Ports | `ss -ltnp`, `nc -zv -w 3 host 443` | who listens / is a port reachable |
| HTTP | `curl -sS -o /dev/null -w '%{http_code} %{time_total}\n' URL` | status and latency |
| HTTP | `curl -v URL`, `curl -fsS URL`, `curl --resolve host:443:IP URL` | debug / fail on 4xx-5xx / test new IP |
| DNS | `dig name +short`, `dig @1.1.1.1 name`, `dig +trace name`, `getent hosts name` | resolve and compare |
| TLS | `openssl s_client -connect h:443 -servername h </dev/null \| openssl x509 -noout -subject -issuer -dates` | certificate details |
| Path | `ping -c 3 host`, `tracepath -n host`, `sudo traceroute -T -p 443 -n host` | reachability and hops |
| SSH | `ssh-keygen -t ed25519 -C id`, `ssh -L 5432:db:5432 bastion` | new key / tunnel |

```text
CIDR:   /16 = 65,536   /20 = 4,096   /24 = 256 (251 usable)   /26 = 64   /28 = 16 (11 usable)   /32 = 1
        addresses = 2^(32 - prefix); private: 10/8, 172.16/12, 192.168/16
curl:   6 DNS   7 refused   22 HTTP>=400 (with -f)   28 timeout   35 TLS handshake   60 bad certificate
Exit:   0 ok   1 failure   2 usage   126 not executable   127 not found   130 Ctrl+C   137 SIGKILL/OOM
HTTP:   2xx ok   3xx redirect   401/403 auth   404 path   429 throttled   502 bad backend   503 no healthy   504 slow
DNS:    NOERROR   NXDOMAIN = no such name   SERVFAIL = resolver could not answer
```
