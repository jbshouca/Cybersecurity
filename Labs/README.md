# Penetration Testing Home Labs

Hands-on penetration testing labs built with VMware Workstation Pro. Each lab progressively builds on the previous one, introducing new tools, techniques, and attack scenarios.

## Lab Environment

| VM | OS | Role |
|---|---|---|
| Kali Linux | Kali 2026.x | Attacker machine |
| Debian | Debian 12+ | DMZ server / pivot host |
| CentOS | CentOS Stream 9 | Middleware / jump box |
| Ubuntu | Ubuntu 26.04 | Internal target |

### Network Layout

```
Primary Network:    192.168.244.0/24  (all VMs)
Internal Network:   172.16.0.0/24     (CentOS + Ubuntu only — requires second NIC)
```

## Labs

| Lab | Focus Areas | Difficulty |
|---|---|---|
| [Lab 1](lab1-foundations.md) | Recon, enumeration, credential discovery, lateral movement, iptables pivoting, privilege escalation | Beginner |
| [Lab 2](lab2-iptables-routing.md) | iptables deep-dive, ip route, SSH tunnels, NAT/masquerade, egress filtering, firewall bypass | Intermediate |
| [Lab 3](lab3-ssh-tunnels-metasploit.md) | SSH local/dynamic/reverse tunnels, Chisel, Metasploit through tunnels, command injection, SOCKS proxying | Intermediate-Advanced |
| [Lab 4](lab4-tunnel-mastery.md) | SSH local/dynamic/reverse forwarding, SSH jump chaining, Metasploit through tunnels, multi-hop pivoting | Advanced |

## Tools Practiced

- **Reconnaissance:** nmap, netcat (nc), dirb, curl
- **Exploitation:** Metasploit (msfconsole, auxiliary scanners, handlers), Hydra, command injection
- **Pivoting:** SSH tunnels (local, dynamic, reverse), Chisel, iptables NAT, ip route
- **Post-Exploitation:** SUID binary exploitation, writable cron jobs, credential harvesting, privilege escalation
- **Monitoring:** tcpdump, iptables logging, proxychains

## Prerequisites

- VMware Workstation Pro installed
- All four VMs built and networked (NAT for primary network)
- CentOS and Ubuntu each have a second NIC on the same Host-only VMnet for the 172.16.0.0/24 network
- Kali has internet access for tool installation
- Basic Linux command-line familiarity

## Concepts Mapped to the Cyber Kill Chain

| Kill Chain Phase | Where It's Practiced |
|---|---|
| Reconnaissance | nmap scanning, nc banner grabbing, web enumeration |
| Weaponization | MSFVenom payload generation, exploit selection |
| Delivery | SSH with stolen creds, command injection payloads |
| Exploitation | Metasploit exploits, command injection, SUID abuse |
| Installation | Reverse shells, persistence via cron |
| Command & Control | SSH tunnels, Chisel SOCKS proxy, Metasploit sessions |
| Actions on Objectives | Reading flags, credential harvesting, data exfiltration |
