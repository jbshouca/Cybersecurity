# Linux Administration
 
Permissions, package management, systemd, processes, and logs.
 
---
 
## File permissions
 
### Reading the descriptor
 
```
-rwxr-xr-x
│├──┤├──┤├──┤
│ │    │    └── Others (everyone else)
│ │    └────── Group (users in the file's group)
│ └────────── Owner
└──────────── File type
```
 
### File types
 
| Symbol | Type |
|---|---|
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |
| `b` | Block device (disk) |
| `c` | Character device (terminal) |
 
### Permission values
 
| Perm | Symbol | Numeric |
|---|---|---|
| Read | `r` | 4 |
| Write | `w` | 2 |
| Execute | `x` | 1 |
| None | `-` | 0 |
 
Example: `-rwxr-xr-x` = owner has rwx (7), group has r-x (5), others have r-x (5) → **755**
 
### chmod
 
```bash
# Numeric mode
chmod 755 file.txt
 
# Symbolic mode
chmod u+x script.sh          # add execute for user
chmod g-w file.txt           # remove write for group
chmod o=r file.txt           # set others to read only
 
# Recursive — apply to directory and everything inside
chmod -R 755 /path/to/dir
```
 
### chown
 
```bash
# Change owner
sudo chown newuser file.txt
 
# Change owner and group
sudo chown newuser:newgroup file.txt
 
# Recursive
sudo chown -R newuser:newgroup /path/to/dir
```
 
---
 
## Package management
 
### apt / dpkg (Debian, Ubuntu, Kali)
 
```bash
# Update package lists
sudo apt update
 
# Upgrade all packages
sudo apt upgrade -y
 
# Install a package
sudo apt install <package> -y
 
# Remove a package
sudo apt remove <package>
 
# Download but do NOT install (useful for air-gapped installs)
sudo apt download <package>
 
# Search
apt search <name>
```
 
**dpkg** is the low-level tool that installs `.deb` files:
 
```bash
# Install a .deb file
sudo dpkg -i package.deb
 
# Fix dependencies afterward
sudo apt install -f -y
 
# Or use apt directly (handles deps automatically)
sudo apt install ./package.deb
 
# Remove
sudo dpkg -r package
 
# Purge (remove + config files)
sudo dpkg -P package
 
# List installed packages
dpkg -l
dpkg -l | grep nmap
 
# Files installed by a package
dpkg -L nmap
 
# Which package owns a file
dpkg -S /usr/bin/nmap
 
# Inspect a .deb without installing
dpkg -c package.deb    # contents
dpkg -I package.deb    # info
```
 
### yum / dnf / rpm (RHEL, CentOS, Fedora)
 
```bash
sudo dnf install <package> -y
sudo dnf update -y
 
# Install an .rpm directly
sudo rpm -ivh package.rpm
```
 
### pip (Python)
 
```bash
# Install pip
sudo apt install python3-pip -y      # Debian/Ubuntu
sudo dnf install python3-pip -y      # CentOS
 
# Install a package
pip3 install requests
 
# Install a .whl (pre-built wheel)
pip3 install package.whl --break-system-packages
 
# Download wheels for offline install later
pip3 download requests
```
 
A `.whl` (wheel) file is a pre-built Python package — the `.deb`/`.rpm` equivalent for Python.
 
### Where package sources live
 
```bash
# Debian/Ubuntu
cat /etc/apt/sources.list
 
# To add a custom repo, put its URL in this file (or in /etc/apt/sources.list.d/)
```
 
---
 
## systemd and systemctl
 
**systemd** is the first process (PID 1) that runs when Linux boots. It manages every service that starts after. **systemctl** is how you talk to systemd.
 
### Common commands
 
```bash
# Start / stop / restart
sudo systemctl start ssh
sudo systemctl stop ssh
sudo systemctl restart ssh
 
# Reload config without full restart (not all services support this)
sudo systemctl reload ssh
 
# Check status
sudo systemctl status ssh
 
# Enable / disable on boot
sudo systemctl enable ssh
sudo systemctl disable ssh
 
# Enable + start together
sudo systemctl enable --now ssh
 
# Disable + stop together
sudo systemctl disable --now ssh
```
 
### Where unit files live
 
| Path | Purpose | Priority |
|---|---|---|
| `/etc/systemd/system/` | Admin overrides and custom services | **Highest** |
| `/run/systemd/system/` | Runtime units (temporary, gone on reboot) | Middle |
| `/lib/systemd/system/` | Package-installed defaults | Lowest |
 
### Checking systemd from a blue-team perspective
 
```bash
# Suspicious services
systemctl list-units --type=service --all | grep -v "loaded active"
 
# What's enabled to start on boot
systemctl list-unit-files --type=service --state=enabled
 
# Failed services (possible crashed exploit or malware)
systemctl --failed
 
# Recently created / modified unit files
find /etc/systemd/system/ -type f -newermt "7 days ago"
find /lib/systemd/system/ -type f -newermt "7 days ago"
 
# User-level services (attackers can create per-user services too)
find /home -path "*/.config/systemd/user/*.service" 2>/dev/null
 
# Recent warnings
journalctl --since "24 hours ago" -p warning
 
# Log for a specific service
journalctl -u <service> -n 100 --no-pager
```
 
---
 
## Processes and resource monitoring
 
### Viewing running processes
 
```bash
# Top — live, sortable process view
top
 
# htop — nicer interactive version
htop
 
# ps aux — full list of all processes
ps aux
 
# Sort by CPU / memory
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
```
 
### Finding a specific process
 
```bash
# By name
ps aux | grep firefox
 
# Avoid grep matching itself
ps aux | grep [f]irefox
# or
ps aux | grep firefox | grep -v grep
 
# Cleaner alternatives
pgrep firefox           # just PIDs
pgrep -a firefox        # PID + command line
pidof firefox           # exact name match
```
 
The `ps aux` output columns:
```
USER  PID  %CPU  %MEM  VSZ  RSS  TTY  STAT  START  TIME  COMMAND
```
 
The **second column is the PID** — this is what you pass to `kill`.
 
### htop navigation
 
- `/` — search (highlights matches)
- `F4` (or `\`) — filter (hides non-matching)
- `F6` — pick sort column
- `P`, `M`, `T` — quick sort by CPU, memory, time
- `F5` (or `t`) — tree view
- `F9` (or `k`) — kill selected process
### Killing processes
 
```bash
# Graceful (SIGTERM)
kill <PID>
kill -15 <PID>
 
# Forceful (SIGKILL)
kill -9 <PID>
 
# By name
pkill firefox
pkill -9 firefox
killall firefox
 
# By full command line, not just name
pkill -f "python3 myscript.py"
 
# By port — what's using port 8080?
sudo lsof -i :8080
```
 
### Signal cheat sheet
 
| Signal | Number | Behavior |
|---|---|---|
| SIGTERM | 15 | Default. Asks nicely. Process can clean up. Try first. |
| SIGKILL | 9 | Kernel kills immediately. No cleanup. Last resort. |
| SIGHUP | 1 | Many daemons treat as "reload config" |
| SIGINT | 2 | Same as Ctrl+C |
 
---
 
## Logged in users and open files
 
```bash
# Currently logged in users
who
 
# All open files (huge — pipe to less)
sudo lsof | less
 
# Open files by user
sudo lsof -u <username>
 
# Network connections by user
sudo lsof -u <username> -i
 
# Files opened by root
sudo lsof -u root
 
# Exclude a user
sudo lsof -u ^root
```
 
---
 
## Where logs live
 
Most logs live under `/var/log/`.
 
### dmesg (kernel messages)
 
Kernel's own message buffer — hardware detection, driver loading, disk errors, memory issues, network interface changes, USB events.
 
```bash
dmesg
dmesg -T              # human-readable timestamps
dmesg -l err          # errors only
dmesg -w              # follow in real time
dmesg | grep -i usb
dmesg | grep -i error
```
 
### syslog
 
Catch-all for system activity — services starting/stopping, cron jobs, application messages, network events. Anything not kernel-specific or auth-specific.
 
### wtmp
 
Binary file recording every **successful login and logout**, plus system reboots and shutdowns.
```bash
last
```
 
### btmp
 
Binary file recording every **failed login attempt** — wrong passwords, invalid usernames, failed SSH.
```bash
lastb
```
 
### auth.log
 
Everything related to authentication and authorization — successful and failed SSH logins, sudo usage, password changes, user account modifications.
 
### journalctl
 
```bash
# All errors and above
sudo journalctl -p err
 
# Errors for a specific service
sudo journalctl -p err -u ssh
sudo journalctl -p err -u networking
 
# Follow errors in real time
sudo journalctl -p err -f
```
 
