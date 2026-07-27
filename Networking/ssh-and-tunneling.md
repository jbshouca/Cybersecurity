# SSH & Tunneling
 
SSH fundamentals, config, all forwarding types, jump hosts, and WireGuard for pivoting.
 
---
 
## What SSH is
 
SSH (Secure Shell) is a protocol that lets you log into and control another computer remotely over an **encrypted** connection. Everything sent is encrypted.
 
Use cases:
- Remote command-line access
- Secure file transfers (SCP, SFTP)
- Port forwarding / tunneling
### How it works
 
1. Client connects to port 22 on the server
2. They negotiate encryption — agree on ciphers, exchange keys
3. Client authenticates (password or key)
4. Access granted to a shell on the remote machine
### Key-based auth
 
`ssh-keygen` creates a **private key** (stays on your machine, never share it) and a **public key** (goes on the server you want to access). When you connect, SSH uses math to prove you have the private key without ever sending it over the network.
 
```bash
# Generate a key
ssh-keygen -t ed25519
 
# Copy your public key to a server
ssh-copy-id user@server_ip
```
 
---
 
## Basic SSH usage
 
```bash
# Connect
ssh user@host
 
# Non-standard port
ssh -p 2222 user@host
 
# Specify a key
ssh -i ~/.ssh/id_ed25519 user@host
 
# IPv6 (link-local — need to specify NIC with %)
ip -6 a | grep fe80
ssh user@fe80::20c:29ff:fe4a:7b2e%ens33
```
 
---
 
## SSH config file
 
`~/.ssh/config` lets you save connection details so you can just type `ssh <name>`.
 
### Example — jump host, aliased hosts, custom port
 
```
# Jump host (Debian)
Host debian-jump
    HostName 192.168.170.131
    User platinum
    IdentityFile ~/.ssh/id_ed25519
 
# Ubuntu — accessed through the jump host
Host ubuntu
    HostName 192.168.170.132
    User user
    ProxyJump debian-jump
    IdentityFile ~/.ssh/id_ed25519
 
# CentOS — direct connection, non-standard port
Host centos
    HostName 192.168.170.133
    User user
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
```
 
Now: `ssh ubuntu` bounces through the Debian jump host automatically.
 
> `IdentityFile` tells SSH which private key to use for that host.
 
---
 
## SSH jump hosts (bastion hosts)
 
A jump host is a machine you hop through to reach another machine you can't access directly. In real networks, sensitive servers aren't exposed to the internet — you first connect to a bastion that has access to both external and internal networks.
 
### One-liner
 
```bash
# Kali → Debian → Ubuntu, in one command
ssh -J user@debian_ip user@ubuntu_ip
 
# Different ports or users
ssh -J user1@host1:port1 user2@host2 -p port2
 
# Multiple jumps
ssh -J user1@host1,user2@host2 user3@host3
```
 
---
 
## Enabling SSH forwarding on the server
 
To use tunneling **through** a machine, its sshd needs to allow it:
 
```bash
sudo vim /etc/ssh/sshd_config
```
Ensure:
```
AllowTcpForwarding yes
GatewayPorts yes
```
Then:
```bash
sudo systemctl restart ssh
```
 
---
 
## SSH tunneling: three types
 
### 1. Local port forwarding — "reach something over there from here"
 
Listen on a port on your machine; anything hitting it goes through the encrypted tunnel to a destination on the other side.
 
```bash
ssh -L 8080:target_server:80 user@ssh_server
```
 
Now `http://localhost:8080` on your machine actually hits `target_server:80`, all encrypted through the tunnel.
 
**Real example:** you have SSH to Debian, and want to reach Ubuntu's web server on port 80:
```bash
ssh -L 8080:192.168.244.130:80 user@192.168.244.132
curl http://localhost:8080
```
 
**Background tunnel (no shell):**
```bash
ssh -fN -L 8080:192.168.170.132:80 platinum@192.168.170.131
```
- `-f` — go to background
- `-N` — don't execute a remote command
### 2. Remote port forwarding — "let someone over there reach something here"
 
The reverse. The remote SSH server listens on a port and forwards back through the tunnel to you.
 
```bash
ssh -R 8080:localhost:80 user@ssh_server
```
 
Now `ssh_server:8080` forwards to your local `:80`.
 
**Attack use case:** compromise an internal machine, SSH out to your external server with `-R`, and now you can access the internal machine from the outside through that tunnel. Most firewalls allow outbound SSH, so this often goes undetected.
 
### 3. Dynamic port forwarding — SOCKS proxy
 
Turns the SSH connection into a general-purpose proxy. Instead of forwarding one port, you get a SOCKS proxy any application can route through.
 
```bash
ssh -D 9050 user@ssh_server
```
 
Now configure your browser (or any app) to use `localhost:9050` as a SOCKS proxy — all its traffic goes through the encrypted tunnel and exits from the SSH server. A poor man's VPN.
 
---
 
## Pivoting scenario walkthrough
 
```
[Your Kali]  →  [Compromised Web Server]  →  [Internal Database Server]
192.168.1.50    10.0.0.5                    10.0.0.20:3306
```
 
Kali cannot reach `10.0.0.20` directly. You've compromised the web server and have SSH creds. Reach the database:
 
```bash
ssh -L 3306:10.0.0.20:3306 user@10.0.0.5
```
 
"Listen on 3306 on my Kali; anything that hits it, tunnel through to the web server, and from there forward to the database on 3306."
 
Now on Kali:
```bash
mysql -h 127.0.0.1 -P 3306 -u dbadmin -p
```
 
---
 
## WireGuard for pivoting
 
WireGuard is a modern VPN that runs in the kernel (not userspace like OpenVPN). It's faster, simpler, and uses non-negotiable modern crypto — no choosing weak ciphers.
 
- Config is minimal
- Codebase is ~4,000 lines (vs. OpenVPN's ~100,000)
### Scenario: pivot through Ubuntu to reach an isolated network
 
**On Ubuntu (the pivot):**
```ini
[Interface]
PrivateKey = UBUNTU_PRIVATE_KEY
Address = 10.0.0.2/24
ListenPort = 51820
 
# Enable IP forwarding + NAT when the tunnel comes up
PostUp = sysctl -w net.ipv4.ip_forward=1; iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
PostDown = iptables -t nat -D POSTROUTING -o ens33 -j MASQUERADE
 
[Peer]
PublicKey = KALI_PUBLIC_KEY
AllowedIPs = 10.0.0.1/32
```
 
**On Kali — route internal traffic through WireGuard:**
```ini
[Interface]
PrivateKey = KALI_PRIVATE_KEY
Address = 10.0.0.1/24
 
[Peer]
PublicKey = UBUNTU_PUBLIC_KEY
AllowedIPs = 10.0.0.2/32, 172.16.0.0/24
Endpoint = UBUNTU_IP:51820
```
 
The key line is `AllowedIPs = 10.0.0.2/32, 172.16.0.0/24` — it tells WireGuard "route traffic for 172.16.0.0/24 through this tunnel too." Combined with the NAT masquerade on Ubuntu, Kali can now reach any host on 172.16.0.0/24 through the tunnel.
 
---
 
## Related — Chisel for tunneling over HTTP
 
When SSH is blocked but HTTP/HTTPS egress is allowed, Chisel fills the gap. See [Post-Exploitation & Pivoting](./post-exploitation-pivoting.md#chisel) for details.
