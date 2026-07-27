# Reference
 
Foundational references — OSI model, Cyber Kill Chain, Linux file structure, regex, tmux, telnet, and cellular tech.
 
---
 
## OSI Model
 
```
Layer | Name         | What it does              | Protocols / Services                        | Data Unit | Devices
------+--------------+---------------------------+---------------------------------------------+-----------+------------------
  7   | Application  | User interaction          | HTTP, HTTPS, FTP, SSH, DNS, SMTP, POP3,     | Data      | Firewalls (L7)
      |              | What you see and use      | IMAP, SNMP, DHCP, Telnet, SMB, LDAP         |           |
------+--------------+---------------------------+---------------------------------------------+-----------+------------------
  6   | Presentation | Translation / format      | SSL/TLS, JPEG, GIF, PNG, ASCII, Unicode,    | Data      |
      |              | Encryption, compression   | MPEG, encryption, compression                |           |
------+--------------+---------------------------+---------------------------------------------+-----------+------------------
  5   | Session      | Connection management     | NetBIOS, RPC, PPTP, SIP, SMB sessions       | Data      |
      |              | Establish, maintain, end  |                                             |           |
------+--------------+---------------------------+---------------------------------------------+-----------+------------------
  4   | Transport    | Reliable delivery         | TCP, UDP                                    | Segment   | Load balancers
      |              | Port numbers live here    | Port numbers (22, 80, 443, 445)             | (TCP)     |
      |              | Segmentation, flow ctrl   |                                             | Datagram  |
      |              |                           |                                             | (UDP)     |
------+--------------+---------------------------+---------------------------------------------+-----------+------------------
  3   | Network      | Routing / addressing      | IP (IPv4, IPv6), ICMP, IPsec                | Packet    | Routers
      |              | IP addresses live here    | OSPF, BGP, RIP                              |           | L3 switches
      |              | Gets packets between nets |                                             |           |
------+--------------+---------------------------+---------------------------------------------+-----------+------------------
  2   | Data Link    | MAC addressing            | Ethernet, Wi-Fi (802.11), ARP               | Frame     | Switches
      |              | Local network delivery    | PPP, VLAN (802.1Q)                          |           | Bridges, NICs
      |              | Same-segment comms        |                                             |           |
------+--------------+---------------------------+---------------------------------------------+-----------+------------------
  1   | Physical     | Bits on the wire          | Ethernet cables, fiber optic                | Bits      | Hubs, repeaters
      |              | Raw signals / electricity | Wi-Fi radio, USB, Bluetooth                 |           | Modems
      |              | Actual hardware           |                                             |           |
```
 
---
 
## Cyber Kill Chain
 
Seven-step framework created by Lockheed Martin that maps out every stage an attacker goes through to carry out a cyberattack. Understanding the framework helps with defense.
 
### 1. Reconnaissance
Attacker researches and selects a target. Gathers information before ever touching the target's network.
- **Passive:** Googling the company, LinkedIn profiles, DNS records. No direct contact.
- **Active:** port scanning, probing systems.
- **Defense:** monitor for port scans, limit public info, train employees.
### 2. Weaponization
Creates the attack tool. Combines an exploit (the thing that breaks in) with a payload (the thing that does the damage).
 
### 3. Delivery
Sends the weapon to the target. Most common vectors:
- Email attachments
- Malicious websites
- USB drives
- **Defense:** email filtering, web proxies, endpoint protection, user training.
### 4. Exploitation
The weapon fires — the vulnerability is triggered, malicious code executes. Three categories:
- Application vulnerability (unpatched software)
- OS vulnerability (unpatched OS)
- The user themselves (social engineering)
- **Defense:** patch management, application whitelisting, EDR, least-privilege access.
### 5. Installation
Installs a persistent backdoor on the victim's system. Goal: maintain access even if the system reboots or the user changes their password.
- **Defense:** EDR, file integrity monitoring, application whitelisting.
### 6. Command and Control (C2)
The installed malware "calls home" — opens a communication channel back to the attacker's infrastructure so they can remotely control the compromised system.
- **Defense:** monitor outbound traffic for unusual patterns, DNS monitoring for suspicious domains, network segmentation, proxy web traffic.
### 7. Actions on Objectives
The actual goal — what the attacker came for:
- Data exfiltration (stealing files)
- Data destruction (ransomware, wiping systems)
- Lateral movement (using the compromised machine to attack other internal systems)
- Privilege escalation (gaining admin/root access)
- Persistence expansion (compromising more machines)
- **Defense:** DLP, network segmentation, monitoring, encryption at rest, incident response plan.
---
 
## Linux file structure
 
```
/
├── bin/         Essential user binaries
├── sbin/        Essential system binaries
├── usr/         Secondary programs and libraries
│   ├── bin/     Most user commands live here
│   ├── sbin/    Most system commands live here
│   ├── lib/     Libraries for /usr/bin and /usr/sbin
│   ├── local/   Manually compiled/installed software
│   │   ├── bin/
│   │   ├── sbin/
│   │   └── lib/
│   └── share/   Architecture-independent data (docs, man pages)
├── opt/         Optionally installed software (third-party, self-contained)
├── etc/         Configuration files
├── home/        User home directories
├── root/        Root user's home directory
├── var/         Variable data (logs, databases, mail, websites)
│   ├── log/     Log files
│   ├── www/     Web server files
│   ├── tmp/     Temporary files that survive reboot
│   └── spool/   Print queues, mail queues, cron
├── tmp/         Temporary files (cleared on reboot)
├── dev/         Device files
├── proc/        Virtual filesystem — running process info
├── sys/         Virtual filesystem — hardware/kernel info
├── mnt/         Temporary mount points
├── media/       Removable media (USB drives, CDs)
├── boot/        Kernel and bootloader files
├── lib/         Essential shared libraries
├── run/         Runtime data (PIDs, sockets)
└── srv/         Service data (FTP, HTTP served files)
```
 
### The three most-hunted directories
 
1. **User binaries** — `/usr/bin/` and `/bin/`. Normal user commands.
2. **System binaries** — `/usr/sbin/` and `/sbin/`. Admin commands (usually need sudo).
3. **Optionally installed** — `/opt/`. Third-party commercial software, manually downloaded tools, self-contained applications.
---
 
## Regex
 
Regex is a pattern language for searching text — foundational for grep, log parsing, Python scripting, SIEM detection rules, and sed/awk pipelines.
 
Since it's used across nearly every topic in this repo, it lives in its own file: **[regex.md](./regex.md)**.
 
Covers: character shortcuts, quantifiers, anchors, character classes, groups, backreferences, lookarounds, practical patterns (IPs, emails, hashes, CVEs, JWTs), and usage with grep, Python, sed, and awk.
 
---
 
## tmux quick reference
 
Every tmux command begins with `Ctrl+B`, then let go and press the second key.
 
### Session management (from outside tmux)
 
```
tmux                        Start new session
tmux new -s name            Start named session
tmux ls                     List sessions
tmux attach                 Reattach to last session
tmux attach -t name         Reattach to named session
tmux kill-session -t name   Kill a session
```
 
### Panes (splitting) — after Ctrl+B
 
```
"           Split horizontal (top/bottom)
%           Split vertical (side by side)
arrow       Move between panes
x           Kill current pane
z           Zoom pane full screen (again to unzoom)
```
 
### Windows (tabs) — after Ctrl+B
 
```
c           Create new window
n           Next window
p           Previous window
,           Rename current window
&           Kill current window
1-9         Jump to window number
```
 
### Copy mode — after Ctrl+B
 
```
[           Enter copy mode
Space       Start selection (in copy mode)
Enter       Copy selection
]           Paste
```
 
---
 
## Telnet
 
Telnet is a plaintext remote terminal protocol from 1969 that opens an interactive session to a remote host over TCP (default port 23). Sends **everything unencrypted**, including credentials.
 
### Original use case (deprecated)
 
Remote login — the way SSH is used today. Nobody should do this anymore; SSH replaced it in the late 90s.
 
### Modern uses (as a raw TCP client)
 
**1. Check if a TCP port is open:**
```bash
telnet target.example.com 80
```
- Connection succeeds → port open
- "Connection refused" → port closed but host up
- Hangs / times out → firewalled or host down
**2. Manually speak text-based protocols (HTTP, SMTP, POP3, IMAP, FTP control):**
```bash
telnet example.com 80
GET / HTTP/1.1
Host: example.com
 
```
(blank line ends the request)
 
**3. Banner grabbing during recon:**
```bash
telnet target 22
```
Prints the SSH banner before disconnect.
 
**4. Talking to network gear** — some old routers/switches still only offer telnet management.
 
### Limitations
 
- **No encryption** — credentials sniffable
- **No integrity protection** — traffic can be modified in transit undetected
- **No key-based auth** — password only
- **No port forwarding / tunneling** — unlike SSH
- **Blocked on most modern networks** outbound
- **Deprecated** for real remote access; SSH is the drop-in replacement
- Only useful as a raw TCP client for **text-based** protocols
### Modern alternatives
 
- **`nc` / `ncat`** — cleaner, scriptable, supports UDP, TLS with ncat
- **`curl`** — for HTTP specifically
- **`openssl s_client`** — for testing TLS-wrapped versions
- **`ssh`** — for actual remote login
```bash
nc -zv target.example.com 80        # port check
nc target.example.com 80            # interactive raw TCP
```
 
---
 
## Cellular technology
 
Cellular tech divides geographic areas into small zones called **cells**, each with its own low-power radio transmitter (cell tower). Instead of one giant transmitter covering an entire city, hundreds of small ones hand off your connection as you move.
 
### GSM (Global System for Mobile Communications)
 
The dominant global standard, used in 220+ countries. Uses **time division** — multiple calls take turns sharing the same radio frequency; each gets a tiny time slot rotated so fast you never notice.
 
- Uses SIM cards
- Weaker in rural areas than CDMA
- Better for international travel
### CDMA (Code Division Multiple Access)
 
Primarily US, used by Verizon and Sprint. Uses **code division** — multiple calls happen on the same frequency simultaneously, each encoded with a unique key. Receiver uses the matching key to decode only the intended call.
 
- Authenticates the device itself rather than a SIM
- Better rural coverage
- Historically couldn't do voice + data simultaneously
### 5G — security-relevant properties
 
- **Software-defined networking.** First all-software network. Previous generations used purpose-built hardware for routing and network management; 5G virtualizes these functions in software. That means the same class of vulnerabilities (buffer overflows, injection, misconfigurations) now applies directly to cellular network infrastructure.
- **Network slicing.** 5G can create virtual isolated network segments (slices) for different purposes — one for IoT, another for emergency services, another for consumer data. If isolation between slices fails, an attacker who compromises one slice could reach another. Conceptually similar to VLAN hopping or VM escape.
- **Massive IoT expansion.** 5G is designed to connect billions of devices — smart cities, autonomous vehicles, medical devices, industrial sensors. Every connected device is an attack surface. Many IoT devices have weak security, no update mechanisms, and default credentials.
 
