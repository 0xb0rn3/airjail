# airjail 🔒

```
  █████╗ ██╗██████╗      ██╗ █████╗ ██╗██╗
 ██╔══██╗██║██╔══██╗     ██║██╔══██╗██║██║
 ███████║██║██████╔╝     ██║███████║██║██║
 ██╔══██║██║██╔══██╗██   ██║██╔══██╗██║██║
 ██║  ██║██║██║  ██║╚█████╔╝██║  ██║██║███████╗
 ╚═╝  ╚═╝╚═╝╚═╝  ╚═╝ ╚════╝ ╚═╝  ╚═╝╚═╝╚══════╝
```

**WiFi Network Manager & Abuse Control — CLI**
> dev: 0xb0rn3 | oxbv1 · v1.3.0

---

## Features

| Key | Feature | Description |
|-----|---------|-------------|
| `1` | **Network Scan** | ARP scan — IP, MAC, vendor, hostname, BW. ARP change detection flags MAC spoofing in real time. |
| `2` | **Device Lookup** | Deep profile — vendor, hostname, RX/TX, MITM status, full DNS query history |
| `3` | **Deauth** | Kick devices off WiFi. Single burst or continuous background loop. LED fires deauth blast pattern on each burst. |
| `4` | **DNS Spy** | Passive DNS sniff — see what sites each user visits. LED double-blink on activate. CSV export. |
| `5` | **Bandwidth Monitor** | Live per-device RX/TX. Set MB/GB threshold to auto-jail overusers. |
| `6` | **Jail / Redirect** | Captive portal redirect via iptables DNAT. Blocks HTTP, HTTPS, DoT (853), known DoH IPs. LED jail flash on each capture. Session persisted to disk. |
| `7` | **Firewall Status** | Live iptables rule viewer (FILTER + NAT) |
| `8` | **Setup** | Auto-detect interface, gateway, configure portal IP, restore saved session |
| `A` | **Adapter / Driver** | USB adapter detection, driver install, full LED control panel |
| `Q` | **Quit** | Flush iptables, restore ARP tables, save session, clean exit |

---

## USB Adapter Engine (v1.3.0)

airjail now ships a full USB wireless adapter engine ported from [mtkdrvlder](https://github.com/0xb0rn3/mtkdrvlder). On startup, airjail automatically scans the USB bus and identifies your adapter.

### Supported Adapters — 60+ entries

| Family | Chipsets | USB IDs (examples) | Driver | LED |
|--------|----------|--------------------|--------|-----|
| MediaTek | MT7612U | `0e8d:7612`, `0e8d:7662`, `7392:b711`, `2357:010c` + 8 more | `mt76x2u` | ✅ |
| MediaTek | MT7610U | `0e8d:7610`, `148f:761a`, `2357:0103` | `mt76x0u` | ✅ |
| MediaTek | MT7601U | `0e8d:7601`, `148f:7601`, `2357:010e` | `mt7601u` | — |
| MediaTek | MT7921U/7922U | `0e8d:7961`, `0e8d:7922`, `13b1:0043` | `mt7921u` | — |
| Ralink | RT5370/RT3070/RT5572 | `148f:5370`, `148f:3070`, `148f:5572` | `rt2800usb` | — |
| Realtek | RTL8812AU | `0bda:8812`, `2357:0101`, `0846:9052` + 7 more | `88XXau` | — |
| Realtek | RTL8812BU | `0bda:b812`, `2357:012d`, `7392:b822` | `88x2bu` | — |
| Realtek | RTL8814AU | `0bda:8813`, `2001:331a`, `0846:9055` | `8814au` | — |
| Realtek | RTL8821CU | `0bda:c811`, `2357:0115`, `7392:c811` | `8821cu` | — |
| Realtek | RTL8188EU/GU | `0bda:f179`, `0bda:8179` | `r8188eu` | — |
| Atheros | AR9271/AR9287 | `0cf3:9271`, `0cf3:7015`, `0846:9030` | `ath9k_htc` | — |

Press `A` from the main menu to detect your adapter, install its driver, trigger LED patterns manually, or view the full database.

---

## LED Engine (MT76 Family)

USB adapters based on MT7612U and MT7610U support hardware LED control via the MT76 debugfs interface at `/sys/kernel/debug/ieee80211/phyN/mt76`. airjail uses register `0x770` (LED_CFG).

### Patterns

| Pattern | Auto-trigger | Description |
|---------|-------------|-------------|
| **Boot Sequence** | Startup (adapter found) | Triple flash → breathing ramp → solid ON |
| **Scan Pulse** | USB scan in adapter menu | Double sonar-ping × 6 |
| **Install Progress** | Driver compilation running | Slow single-pulse background loop |
| **Success** | Driver install complete | 5 rapid + 3 medium flashes → solid ON |
| **Error / SOS** | Driver install failed | SOS morse `··· — — — ···` → OFF |
| **Deauth Blast** | Deauth packet burst fired | Machine-gun × 15 rapid blinks |
| **MITM Alert** | ARP MITM session opened | Fast double-strobe × 8 → HW blink |
| **DNS Start** | DNS Spy activated | Quick double-blink |
| **Jail Flash** | Device jailed | Aggressive triple-flash |
| **Unjail Pulse** | Device released | Gentle double long-pulse |
| **Monitor Heartbeat** | Monitor mode active | Slow two-beat heartbeat loop |

LED state is shown in the status bar at the bottom of the main menu.

### Driver Installation Methods

| Method | Chipset | Process |
|--------|---------|---------|
| `kernel` | MT7601U, RT2800*, RTL8188EU, AR9271, MT7921U | `modprobe` — in-tree |
| `morrownr` | MT7612U, MT7610U | In-kernel first; DKMS git build fallback |
| `rtl8812au` | RTL8812AU | aircrack-ng fork + DKMS |
| `rtl8812bu` | RTL8812BU | morrownr/88x2bu + DKMS |
| `rtl8814au` | RTL8814AU | morrownr/8814au + DKMS |
| `rtl8821cu` | RTL8821CU | morrownr/8821cu + DKMS |
| `ath9k_htc` | AR9271/AR9287 | firmware-atheros + modprobe |

Build directory: `/tmp/airjail_drv_build/`

---

## Supported Distros

| Distro Family | Package Manager |
|---------------|----------------|
| Arch / Manjaro / EndeavourOS / Garuda | `pacman` |
| Debian / Ubuntu / Kali / Parrot / Mint | `apt` |
| Fedora / RHEL 8+ / CentOS Stream / Rocky | `dnf` |
| RHEL 7 / CentOS 7 (legacy) | `yum` |
| openSUSE Leap / Tumbleweed | `zypper` |
| Alpine Linux | `apk` |
| Void Linux | `xbps-install` |
| Gentoo | `emerge` |
| Solus | `eopkg` |

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

To uninstall:
```bash
sudo ./install uninstall
```

---

## Manual Dependencies

**System packages:**
```bash
# Arch / Manjaro
sudo pacman -S python nmap aircrack-ng iptables iw net-tools libpcap dkms linux-headers

# Debian / Ubuntu / Kali
sudo apt install python3 python3-pip nmap aircrack-ng iptables iw net-tools libpcap-dev dkms linux-headers-$(uname -r)

# Fedora / RHEL 8+
sudo dnf install python3 python3-pip nmap aircrack-ng iptables iw net-tools libpcap-devel dkms kernel-devel

# openSUSE
sudo zypper install python3 nmap aircrack-ng iptables iw net-tools libpcap-devel dkms kernel-devel

# Alpine
sudo apk add python3 py3-pip nmap aircrack-ng iptables iw net-tools libpcap-dev dkms
```

**Python packages:**
```bash
pip install rich scapy requests manuf --break-system-packages
```

---

## Deauth Setup

Deauth requires a **second USB WiFi adapter** in monitor mode. Do NOT use your primary interface.

```bash
sudo airmon-ng check kill
sudo airmon-ng start wlan1     # creates wlan1mon
```

Then in airjail → option `3` → enter `wlan1mon` as the monitor interface.

> MT7612U / MT7610U adapters fire the LED **deauth blast** pattern (machine-gun × 15) on every burst.

---

## Captive Portal / Jail

airjail ships with a built-in captive portal (`index.html`).

```bash
sudo airjail-portal          # serves on port 80
sudo airjail-portal 8080     # or any port
```

When jailed, the device's traffic is redirected via `iptables DNAT`. Bypass vectors blocked:
- DNS-over-TLS (port 853)
- Cloudflare DoH (1.1.1.1)
- Google DoH (8.8.8.8, 8.8.4.4)
- All non-port-80 TCP traffic

Customise jail reason:
```
http://<portal-ip>/?reason=Bandwidth+limit+exceeded
```

---

## ARP MITM

Two-direction ARP poison: your machine sits between target and gateway. LED fires **MITM alert** (strobe → hardware blink) when a session opens. ARP tables restored automatically on stop.

```bash
# Combine with
sudo tcpdump -i wlan0 -w capture.pcap host <target-ip>
sudo sslstrip -l 8080
```

---

## Session Persistence

Jailed devices persisted to `/var/lib/airjail/session.json`, restored on next launch.

```bash
sudo rm /var/lib/airjail/session.json   # clear state
```

---

## Port Scanner

Option `9` or `[P]` from network scan — nmap wrapper with presets:

| Preset | Flags | Use |
|--------|-------|-----|
| 1 | `-sV -T4 --open -p-` | Full 65535 port + version |
| 2 | `-sV -T4 --open --top-ports 1000` | Top 1000 (faster) |
| 3 | `-sV -sC -T4 --open -p-` | Full + NSE scripts |
| 4 | `-sU -T4 --top-ports 100` | Top 100 UDP |
| 5 | Custom | Any nmap flags |

---

## File Structure

```
airjail/
├── airjail          # main Python script (v1.3.0)
├── install          # universal installer / uninstaller (9 PMs)
├── index.html       # captive portal page
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
