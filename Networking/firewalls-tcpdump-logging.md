# Firewalls, tcpdump & Logging
 
iptables, tcpdump, rsyslog, and IP forwarding.
 
---
 
## iptables
 
iptables is the command-line tool for setting rules on how Linux handles network traffic — what gets allowed in, what gets blocked, what gets forwarded. It's the interface to the kernel's **netfilter** framework, which does the actual packet filtering.
 
### Tables
 
Tables define the type of operation on packets. Four exist:
 
1. **filter** — allow or block traffic
2. **nat** — change source/destination addresses (port forwarding, masquerading, routing to other hosts)
3. **mangle** — alter packet headers (changing TTL, marking packets for routing)
4. **raw** — bypass connection tracking (exempt traffic from stateful inspection)
### Chains
 
Chains are checkpoints where packets get inspected. Each table has specific chains:
 
1. **INPUT** — packet is arriving and destined for this machine
2. **OUTPUT** — packet was created by this machine and is leaving
3. **FORWARD** — packet is passing through this machine to somewhere else
4. **PREROUTING** — packet just arrived, before any routing decision
5. **POSTROUTING** — packet is about to leave, after routing decision
### How packets flow
 
When a packet hits a chain, iptables checks it against every rule in order, top to bottom. The moment it matches a rule with a terminating target (`ACCEPT`, `DROP`, `REJECT`), it stops — no further rules are checked. If it reaches the end without matching anything, the chain's default policy decides (default is `ACCEPT`).
 
**Rule order matters.**
 
---
 
### Essential commands
 
**Block an IP:**
```bash
iptables -A INPUT -s 59.45.175.62 -j DROP
```
- `-A` — append to chain
- `-s` — source IP
- `-j` — jump to target
**Block an IP range:**
```bash
iptables -A INPUT -s 59.45.175.0/24 -j DROP
```
 
**Block outbound to an IP:**
```bash
iptables -A OUTPUT -d 31.13.78.35 -j DROP
```
 
**Block a specific port:**
```bash
iptables -A INPUT -p tcp -m tcp --dport 22 -s 59.45.175.0/24 -j DROP
```
- `-p tcp` — protocol is TCP
- `-m tcp` — load the TCP module
- `--dport 22` — destination port 22 (SSH)
**Block multiple ports at once:**
```bash
iptables -A INPUT -p tcp -m multiport --dports 22,80,443 -j DROP
```
 
**List all rules:**
```bash
iptables -L -n --line-numbers
```
- `-L` — list
- `-n` — don't resolve DNS
- `--line-numbers` — show position numbers
**Delete a rule:**
```bash
iptables -D INPUT -s 59.45.175.62 -j DROP
iptables -D INPUT 2
```
When deleting multiple rules by line number, **delete highest numbers first** — otherwise the numbers shift and you delete the wrong rules.
 
**Insert a rule at a specific position:**
```bash
iptables -I INPUT 1 -s 59.45.175.10 -j ACCEPT
```
Inserts at line 1 (top), pushing everything else down. Critical for whitelisting an IP inside a blocked range.
 
---
 
### Connection tracking (conntrack)
 
If you block incoming traffic from an IP, you also can't connect to that IP anymore. Your outgoing request reaches them, but their response comes back through the INPUT chain and gets dropped.
 
**Fix:** always allow packets that belong to connections you already started.
```bash
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```
Make this the **first rule** in your INPUT chain.
 
Then you can switch to whitelist mode:
```bash
iptables -P INPUT DROP
```
Everything incoming is now dropped unless a rule explicitly allows it. **Only do this after** adding the conntrack ESTABLISHED,RELATED rule — otherwise your connections will break.
 
> **ESTABLISHED,RELATED explained:** A stateful firewall rule that permits incoming packets belonging to connections already authorized or initiated by the host. Return traffic for legitimate sessions isn't dropped by a default-deny policy, enabling two-way communication without explicit rules for every return packet.
 
---
 
### Negating conditions
 
The `!` operator flips a condition:
```bash
# Drop everything EXCEPT 22, 80, 443
iptables -A INPUT -p tcp -m multiport ! --dports 22,80,443 -j DROP
```
 
### Custom chains
 
Instead of cramming everything into INPUT, create your own chains for organization:
 
```bash
# Create the chain
iptables -N ssh-rules
 
# Add rules to it
iptables -A ssh-rules -s 192.168.170.0/24 -j ACCEPT
iptables -A ssh-rules -j DROP
 
# Reference it from INPUT
iptables -A INPUT -p tcp --dport 22 -j ssh-rules
```
Any SSH traffic hitting INPUT jumps to `ssh-rules` for processing.
 
---
 
### Saving and restoring rules
 
iptables rules disappear on reboot unless saved.
```bash
iptables-save > /tmp/iptables.rules
iptables-restore < /tmp/iptables.rules
```
 
**Persistence across reboots:**
 
CentOS:
```bash
sudo dnf install iptables-services -y
sudo systemctl enable iptables
sudo service iptables save
```
 
Ubuntu/Debian:
```bash
sudo apt install iptables-persistent -y
# Saves automatically during install
sudo netfilter-persistent save    # re-save later
```
 
---
 
### Full lockdown recipe — block everything except SSH
 
```bash
# 1. Flush existing rules
sudo iptables -F
sudo iptables -t nat -F
 
# 2. Reset default policies to accept (temporarily)
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
 
# 3. Verify
sudo iptables -L -n -v
 
# 4. Rules
# Rule 1: Allow loopback (machine talking to itself)
sudo iptables -A INPUT -i lo -j ACCEPT
 
# Rule 2: Allow established/related connections
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
 
# Rule 3: Allow SSH inbound
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
 
# Rule 4: Drop invalid packets
sudo iptables -A INPUT -m conntrack --ctstate INVALID -j DROP
 
# Rule 5: Set default policy — drop everything else
sudo iptables -P INPUT DROP
 
# Allow all outbound (so you can still reach the internet)
sudo iptables -P OUTPUT ACCEPT
```
 
> **IMPORTANT:** `iptables -F` flushes rules only. `-P` sets policy. They're separate. If you set policy to DROP and then flush, you end up with no rules and a default DROP — blocking everything.
 
---
 
### NAT masquerading — subnet-to-subnet routing
 
Make traffic from one subnet look like it comes from another subnet's IP when it reaches the destination:
 
```bash
# Enable IP forwarding first
sudo sysctl -w net.ipv4.ip_forward=1
 
# Allow forwarding between the two subnets
sudo iptables -A FORWARD -s 192.168.170.0/24 -d 172.16.0.0/24 -j ACCEPT
sudo iptables -A FORWARD -s 172.16.0.0/24 -d 192.168.170.0/24 -j ACCEPT
 
# NAT — masquerade traffic leaving toward 172.16.0.0/24
# Rewrites source IP to the outgoing interface's address
sudo iptables -t nat -A POSTROUTING -s 192.168.170.0/24 -o ens36 -j MASQUERADE
```
 
The masquerade rule uses the nat table (`-t nat`) and POSTROUTING chain (after the routing decision, right before the packet leaves the interface).
 
---
 
### Logging dropped traffic
 
```bash
# Log anything not allowed on INPUT
sudo iptables -A INPUT -m limit --limit 5/min --limit-burst 10 \
    -j LOG --log-prefix "DROPPED-INPUT: " --log-level 4
 
# Log anything not allowed on FORWARD
sudo iptables -A FORWARD -m limit --limit 5/min --limit-burst 10 \
    -j LOG --log-prefix "DROPPED-FORWARD: " --log-level 4
 
# Then drop
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT
```
 
The rate limit (`5/min --limit-burst 10`) prevents log flooding.
 
---
 
## IPv4 forwarding
 
Check current state:
```bash
cat /proc/sys/net/ipv4/ip_forward
# 0 = disabled, 1 = enabled
```
 
Enable for this session (doesn't persist):
```bash
sudo sysctl -w net.ipv4.ip_forward=1
# or
echo 1 > /proc/sys/net/ipv4/ip_forward
```
 
Persist across reboot:
```bash
sudo vim /etc/sysctl.conf
# Ensure this line is present:
net.ipv4.ip_forward = 1
 
# Apply without rebooting
sudo sysctl -p
```
 
Or append it in one shot:
```bash
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```
 
---
 
## tcpdump
 
A command-line packet sniffer. Captures traffic passing through your network interfaces so you can read it, filter it, and save it for later analysis.
 
### Basic usage
 
```bash
# Capture everything on the default interface
sudo tcpdump
 
# Specific interface
sudo tcpdump -i eth0
sudo tcpdump -i ens33
 
# List available interfaces
sudo tcpdump -D
 
# Limit packet count
sudo tcpdump -c 50
 
# Show packet contents in ASCII
sudo tcpdump -A
 
# Hex AND ASCII
sudo tcpdump -XX
 
# Don't resolve hostnames (faster, cleaner)
sudo tcpdump -n
 
# Don't resolve hostnames OR port names
sudo tcpdump -nn
```
 
### BPF filters (Berkeley Packet Filter)
 
**By host:**
```bash
sudo tcpdump -nn host 192.168.170.131
sudo tcpdump -nn src host 192.168.170.131
sudo tcpdump -nn dst host 192.168.170.131
```
 
**By port:**
```bash
sudo tcpdump -nn port 22
sudo tcpdump -nn port 80 or port 443
sudo tcpdump -nn src port 22
sudo tcpdump -nn dst port 80
```
 
**By protocol:**
```bash
sudo tcpdump -nn tcp
sudo tcpdump -nn udp
sudo tcpdump -nn icmp
```
 
### Save and read captures
 
```bash
# Save to pcap (open in Wireshark later)
sudo tcpdump -nn -w capture.pcap
 
# Save with packet limit
sudo tcpdump -nn -c 1000 -w capture.pcap
 
# Read a saved capture
sudo tcpdump -nn -r capture.pcap
 
# Read + filter
sudo tcpdump -nn -r capture.pcap port 80
 
# Read + show contents
sudo tcpdump -nn -A -r capture.pcap
```
 
### Troubleshooting connectivity
 
```bash
sudo tcpdump -i ens33 -nn host REMOTE_IP
```
 
**SSH not installed/running on target:**
```
10:30:01 IP src.54321 > tgt.22: Flags [S]
10:30:01 IP tgt.22 > src.54321: Flags [R.]
```
SYN goes out, RST comes back. Port is closed — nothing listening on 22. Fix: start ssh on target.
 
**Firewall silently dropping:**
```
10:30:01 IP src.54321 > tgt.22: Flags [S]
10:30:04 IP src.54321 > tgt.22: Flags [S]
10:30:07 IP src.54321 > tgt.22: Flags [S]
```
SYN goes out, nothing comes back, SYN retransmits. The firewall is silently dropping packets. Fix: allow port 22 on target.
 
---
 
## Remote logging with rsyslog
 
Instead of every machine keeping logs locally, send them to a central log server.
 
### On the log server (receiver)
 
1. Install rsyslog:
```bash
   sudo apt install rsyslog -y
```
2. Enable UDP/TCP modules in `/etc/rsyslog.conf` (uncomment):
```
   module(load="imudp")
   input(type="imudp" port="514")
 
   module(load="imtcp")
   input(type="imtcp" port="514")
```
3. Add template to separate remote logs (before existing rules):
```
   # Separate log files per remote host
   $template RemoteLogs,"/var/log/remote/%HOSTNAME%/%PROGRAMNAME%.log"
   # Store remote logs using the template, stop processing so they don't also
   # go into local syslog
   if $fromhost-ip != '127.0.0.1' then ?RemoteLogs
   & stop
```
4. Create the log directory:
```bash
   sudo mkdir -p /var/log/remote
   sudo chown root:adm /var/log/remote
```
5. Allow port 514 through the firewall:
```bash
   sudo iptables -A INPUT -p tcp --dport 514 -j ACCEPT
   sudo iptables -A INPUT -p udp --dport 514 -j ACCEPT
```
6. Restart rsyslog.
### On each client (sender)
 
Add to the bottom of `/etc/rsyslog.conf`:
```
*.* @@<log_server_ip>:514
```
(`@@` is TCP; single `@` is UDP.)
 
Set the hostname so logs are labeled correctly:
```bash
hostname
sudo hostnamectl set-hostname <name>
```
 
Restart rsyslog and test:
```bash
logger "test log message"
```
 
### Verify connectivity
 
```bash
nc -zv <log_server_ip> 514
```
 
