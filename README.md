# Cybersecurity Notes
 
Personal reference and study notes covering Linux and Windows administration, networking, wireless, reconnaissance, exploitation, and post-exploitation techniques.
 
These notes are organized by topic. Each file is a self-contained reference for that area — commands, concepts, and workflows I've used or want to remember.
 
> **Disclaimer:** All offensive material here is for authorized labs, CTFs, and personal study environments only.
 
---
 
## Contents
 
### Foundations
- [Linux Fundamentals](Foundations/linux-fundamentals.md) — Shell basics, bash scripting, cron, common commands
- [Python for Security](Foundations/python-for-security.md) — Python for scripting, port scanning, reverse shells, log parsing, packet crafting
- [Reference](Foundations/reference.md) — OSI model, Cyber Kill Chain, Linux file structure, tmux, telnet, cellular
### Utility
- [Regex](Utility/regex.md) — Pattern language used across grep, Python, sed/awk, SIEMs
### Linux
- [Linux Administration](Linux/linux-administration.md) — Permissions, packages, systemd, logs, processes
- [Linux Storage & Boot](Linux/linux-storage-and-boot.md) — Partitioning, mounting, fstab, fsck, chroot, GRUB, venv
### Networking
- [Networking Basics](Networking/networking-basics.md) — Interfaces, IP, ping/traceroute/mtr, netstat, netcat
- [Wireless (802.11)](Networking/wireless-802-11.md) — Wi-Fi standards, WPA/WPA2/WPA3, aircrack-ng, handshake capture, PMKID, evil twin
- [SSH & Tunneling](Networking/ssh-and-tunneling.md) — SSH config, jump hosts, local/remote/dynamic forwarding, WireGuard
- [Firewalls, tcpdump & Logging](Networking/firewalls-tcpdump-logging.md) — iptables, tcpdump, rsyslog, IP forwarding
### Windows
- [Windows Administration](Windows/windows-administration.md) — CLI, PowerShell, icacls, firewall, PsExec, Task Scheduler, Event Viewer
### Offensive Security
- [Reconnaissance & Enumeration](Offensive%20Security/reconnaissance-enumeration.md) — nmap, enum4linux, JAWS, searchsploit
- [Exploitation](Offensive%20Security/exploitation.md) — Metasploit, MSFVenom, memory corruption, zero-day concepts
- [Post-Exploitation & Pivoting](Offensive%20Security/post-exploitation-pivoting.md) — PEASS-ng, Chisel, reverse shells, iptables pivoting
### Labs
- [Labs](Labs/readme.md/) — Hands-on lab walkthroughs (In Progress)
---

## Recommended reading order (if new)
 
1. `linux-fundamentals.md`
2. `linux-administration.md`
3. `python-for-security.md`
4. `regex.md`
5. `networking-basics.md`
6. `ssh-and-tunneling.md`
7. `firewalls-tcpdump-logging.md`
8. `wireless-802-11.md`
9. `reconnaissance-enumeration.md`
10. `exploitation.md`
11. `post-exploitation-pivoting.md`
---
