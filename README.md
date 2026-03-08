# airjail 🔒

```
  █████╗ ██╗██████╗      ██╗ █████╗ ██╗██╗
 ██╔══██╗██║██╔══██╗     ██║██╔══██╗██║██║
 ███████║██║██████╔╝     ██║███████║██║██║
 ██╔══██║██║██╔══██╗██   ██║██╔══██║██║██║
 ██║  ██║██║██║  ██║╚█████╔╝██║  ██║██║███████╗
 ╚═╝  ╚═╝╚═╝╚═╝  ╚═╝ ╚════╝ ╚═╝  ╚═╝╚═╝╚══════╝
```

**WiFi Network Manager & Abuse Control — CLI**
> dev: 0xb0rn3 | oxbv1 · v1.2.0

---

## Features

| # | Feature | Description |
|---|---------|-------------|
| 1 | **Network Scan** | ARP scan — discover all LAN devices with IP, MAC, vendor, hostname, BW usage. ARP change detection flags MAC spoofing in real time. |
| 2 | **Device Lookup** | Deep profile of any device — vendor, hostname, RX/TX, MITM status, full DNS query history |
| 3 | **Deauth** | Kick any device off WiFi using deauth frames. Single burst or continuous background loop with configurable interval. |
| 4 | **DNS Spy** | Passively sniff DNS — see what websites each user visits in real time. Export full history to CSV. |
| 5 | **Bandwidth Monitor** | Live per-device RX/TX with accurate iptables counter parsing. Set a MB/GB threshold to auto-jail devices that exceed it. |
| 6 | **Jail / Redirect** | Redirect abusive users to captive portal via iptables DNAT. Blocks HTTP, HTTPS, DNS-over-TLS (853), and known DoH IPs. Session state persisted to disk. |
| 7 | **Firewall Status** | Live iptables rule viewer (FILTER + NAT tables) |
| 8 | **Setup** | Auto-detect interface, gateway, configure portal IP, restore saved session |
| 9 | **Port Scan** | nmap wrapper — 4 presets (full, top-1000, scripts, UDP) or custom flags. Results rendered in a Rich table. |
| 0 | **ARP MITM** | ARP poison both directions (target ↔ gateway). Traffic flows through your machine. Multiple concurrent sessions. Clean ARP restore on stop. |

---

## Supported Distros

| Distro Family | Package Manager | Auto-Install |
|---------------|----------------|-------------|
| Arch / Manjaro / EndeavourOS / Garuda / SteamOS | `pacman` | ✅ |
| Debian / Ubuntu / Kali / Parrot / Pop!_OS / Mint / Raspbian | `apt` | ✅ |
| Fedora / RHEL 8+ / CentOS Stream / AlmaLinux / Rocky | `dnf` | ✅ |
| RHEL 7 / CentOS 7 (legacy) | `yum` | ✅ |
| openSUSE Leap / Tumbleweed / SLES | `zypper` | ✅ |
| Alpine Linux | `apk` | ✅ |
| Void Linux | `xbps-install` | ✅ |
| Gentoo | `emerge` | ✅ |
| Solus | `eopkg` | ✅ |

> The installer auto-enables extra repos where needed (RPM Fusion on Fedora, Alpine community, openSUSE Packman) to resolve `aircrack-ng`.

---

## Quick Install

```bash
git clone https://github.com/0xb0rn3/airjail
cd airjail
chmod +x install
sudo ./install
```

Then run:
```bash
sudo airjail
```

Or directly:
```bash
sudo python3 airjail
```

To uninstall:
```bash
sudo ./install uninstall
```

> airjail auto-installs all missing dependencies on first run as well.

---

## Manual Dependencies

**System packages:**
```bash
# Arch / Manjaro
sudo pacman -S python nmap aircrack-ng iptables iw net-tools libpcap

# Debian / Ubuntu / Kali
sudo apt install python3 python3-pip nmap aircrack-ng iptables iw net-tools libpcap-dev

# Fedora / RHEL 8+
sudo dnf install python3 python3-pip nmap aircrack-ng iptables iw net-tools libpcap-devel

# RHEL 7 / CentOS 7
sudo yum install python3 python3-pip nmap aircrack-ng iptables iw net-tools libpcap-devel

# openSUSE
sudo zypper install python3 nmap aircrack-ng iptables iw net-tools libpcap-devel

# Alpine
sudo apk add python3 py3-pip nmap aircrack-ng iptables iw net-tools libpcap-dev

# Void
sudo xbps-install python3 nmap aircrack-ng iptables iw net-tools libpcap-devel

# Solus
sudo eopkg install python3 nmap aircrack-ng iptables iw net-tools
```

**Python packages (all distros):**
```bash
pip install rich scapy requests manuf --break-system-packages
```

---

## Deauth Setup

Deauth requires a **second USB WiFi adapter** in monitor mode. Do NOT use your primary adapter — it will lose connectivity.

```bash
sudo airmon-ng check kill
sudo airmon-ng start wlan1     # creates wlan1mon
```

Then in airjail → option `3` → enter `wlan1mon` as the monitor interface.

**Continuous deauth mode:** airjail can run deauth in a background loop with a configurable burst interval instead of a single burst. The target stays kicked until you release it from the Jail menu.

---

## Captive Portal / Jail Setup

airjail ships with a built-in captive portal page (`index.html`) — a branded warning/acknowledgement form that jailed devices are redirected to.

**Option A — use the built-in portal launcher (easiest):**
```bash
sudo airjail-portal          # serves index.html on port 80
sudo airjail-portal 8080     # or any other port
```

**Option B — manual:**
```bash
cd /path/to/airjail
python3 -m http.server 80
```

**Option C — any web server (nginx, apache, caddy)** pointed at the airjail directory.

Then in airjail → option `6` → set the portal IP/port → jail the user.

When jailed, the device's HTTP (port 80) and HTTPS (port 443) traffic is redirected to your portal via `iptables DNAT`. The following bypass vectors are also blocked:
- DNS-over-TLS on port 853
- Cloudflare DoH (1.1.1.1)
- Google DoH (8.8.8.8, 8.8.4.4)
- All other TCP traffic (non-port-80)

**Customise the jail reason from the URL:**
```
http://<portal-ip>/?reason=Bandwidth+limit+exceeded
```
The portal page reads the `?reason=` query parameter and displays it as the restriction reason.

---

## ARP MITM

The MITM module (option `0`) performs classic two-direction ARP poisoning between a target and the gateway. Your machine sits in the middle — all traffic passes through it while the target stays online.

```
target ← "gateway is at OUR MAC"
gateway ← "target is at OUR MAC"
```

IP forwarding is kept enabled so connectivity is maintained. On stop, ARP tables are restored automatically (5 gratuitous ARP restore packets sent to both parties).

Combine with:
```bash
sudo tcpdump -i wlan0 -w capture.pcap host <target-ip>
sudo wireshark
sudo sslstrip -l 8080    # HTTP downgrade (HTTP targets only)
```

Multiple targets can be under MITM simultaneously. All sessions are visible in the Network Scan table and the status bar.

---

## Bandwidth Auto-Jail

In the Bandwidth Monitor (option `5`), set a threshold like `500MB` or `2GB`. Any device that exceeds it is automatically jailed and redirected to the captive portal — no manual intervention needed.

```
Auto-jail threshold : 1GB
→ 192.168.1.45 (aa:bb:cc:dd:ee:ff) exceeded limit (1.04 GB) → JAILED
```

Requires the portal IP to be configured first (Setup → option `8`, or Jail menu → option `1`).

---

## Session Persistence

Jailed devices are saved to `/var/lib/airjail/session.json` every time the jail list changes (jail, release, release-all). On next startup, the previous jail state is automatically restored.

```bash
cat /var/lib/airjail/session.json
```

To clear saved state:
```bash
sudo rm /var/lib/airjail/session.json
```

---

## Port Scanner

Option `9` wraps nmap with four presets:

| Preset | Flags | Use Case |
|--------|-------|----------|
| 1 | `-sV -T4 --open -p-` | Full 65535-port + service version |
| 2 | `-sV -T4 --open --top-ports 1000` | Top 1000 (faster) |
| 3 | `-sV -sC -T4 --open -p-` | Full + default NSE scripts |
| 4 | `-sU -T4 --top-ports 100` | Top 100 UDP ports |
| 5 | Custom | Enter any nmap flags manually |

Results are parsed and rendered in a Rich table showing port, state, service, and version. You can also trigger a quick port scan directly from the Network Scan results table with `[P]`.

---

## ARP Spoofing Detection

Every network scan compares results against the known IP→MAC table. If a device's MAC address changes between scans, airjail prints a bold red warning:

```
[⚠] ARP CHANGE DETECTED: 192.168.1.1 was aa:bb:cc:dd:ee:ff → now 11:22:33:44:55:66
```

This catches ARP poisoning attacks against your own machine in real time.

---

## DNS Spy

Option `4` passively sniffs UDP port 53 on your LAN interface — no ARP poisoning required. Per-device query history is kept in memory (up to 500 entries per device) and is also shown in Device Lookup.

Export full history to CSV:
```
DNS Spy → [E] → /tmp/airjail_dns_export.csv
```

---

## Usage Notes

- Must run as **root** — raw sockets + iptables require it
- DNS Spy is **passive** — sniffs port 53 on your LAN interface, no MITM needed
- Bandwidth monitor uses iptables FORWARD chain byte counters — resets when airjail exits
- Jailed state **persists** across restarts via `/var/lib/airjail/session.json`
- On exit (Ctrl+C or `Q`), airjail flushes all iptables rules it created and saves session
- MITM sessions restore ARP tables automatically when stopped
- Vendor MAC lookups are cached in-memory — no repeated API calls during a session

---

## File Structure

```
airjail/
├── airjail          # main Python script
├── install          # universal installer / uninstaller
├── index.html       # built-in captive portal page
├── requirements.txt # Python dependencies
└── README.md
```

---

## Legal

> Only use airjail on networks **you own** or have **explicit written permission** to manage.
> Unauthorized interception, disruption, or redirection of network traffic is illegal in most jurisdictions.

---

## Author

**0xb0rn3 | oxbv1**
GitHub: [github.com/0xb0rn3/airjail](https://github.com/0xb0rn3/airjail)
