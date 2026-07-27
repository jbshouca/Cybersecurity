# Cybersecurity Notes
 
Personal reference and study notes covering Linux and Windows administration, networking, reconnaissance, exploitation, and post-exploitation techniques.
 
These notes are organized by topic. Each file is a self-contained reference for that area — commands, concepts, and workflows I've used or want to remember.
 
> **Disclaimer:** All offensive material here is for authorized labs, CTFs, and personal study environments only.
 
---
 
## Contents
 
### Foundations
- [Linux Fundamentals](./linux-fundamentals.md) — Shell basics, bash scripting, cron, common commands
- [Reference](./reference.md) — OSI model, Cyber Kill Chain, Linux file structure, regex, tmux, telnet, cellular
### Linux
- [Linux Administration](./linux-administration.md) — Permissions, packages, systemd, logs, processes
- [Linux Storage & Boot](./linux-storage-and-boot.md) — Partitioning, mounting, fstab, fsck, chroot, GRUB, venv
### Networking
- [Networking Basics](./networking-basics.md) — Interfaces, IP, ping/traceroute/mtr, netstat, netcat
- [SSH & Tunneling](./ssh-and-tunneling.md) — SSH config, jump hosts, local/remote/dynamic forwarding, WireGuard
- [Firewalls, tcpdump & Logging](./firewalls-tcpdump-logging.md) — iptables, tcpdump, rsyslog, IP forwarding
### Windows
- [Windows Administration](./windows-administration.md) — CLI, PowerShell, icacls, firewall, PsExec, Task Scheduler, Event Viewer
### Offensive Security
- [Reconnaissance & Enumeration](./reconnaissance-enumeration.md) — nmap, enum4linux, JAWS, searchsploit
- [Exploitation](./exploitation.md) — Metasploit, MSFVenom, memory corruption, zero-day concepts
- [Post-Exploitation & Pivoting](./post-exploitation-pivoting.md) — PEASS-ng, Chisel, reverse shells, iptables pivoting
### Labs
- [Labs](./labs/) — Hands-on lab walkthroughs
---
 
## How this repo is organized
 
Each topic file follows a consistent structure:
 
- **Concept** — what it is and why it matters
- **Commands / syntax** — the actual usage, in code blocks
- **Workflow / examples** — how it fits into real tasks
- **Gotchas** — things that trip me up
## Recommended reading order (if new)
 
1. `linux-fundamentals.md`
2. `linux-administration.md`
3. `networking-basics.md`
4. `ssh-and-tunneling.md`
5. `firewalls-tcpdump-logging.md`
6. `reconnaissance-enumeration.md`
7. `exploitation.md`
8. `post-exploitation-pivoting.md`
---
 
