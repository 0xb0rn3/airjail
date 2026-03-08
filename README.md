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
> dev: 0xb0rn3 | oxbv1 · v1.1.0

---

## Features

| # | Feature | Description |
|---|---------|-------------|
| 1 | **Network Scan** | ARP scan — discover all LAN devices with IP, MAC, vendor, hostname, BW usage |
| 2 | **Device Lookup** | Deep profile of any device + full DNS query history |
| 3 | **Deauth** | Kick any device off WiFi using deauth frames |
| 4 | **DNS Spy** | Passively sniff DNS — see what websites each user visits in real time |
| 5 | **Bandwidth Monitor** | Live per-device RX/TX data usage since tool started |
| 6 | **Jail / Redirect** | Redirect abusive users to captive portal via iptables DNAT |
| 7 | **Firewall Status** | Live iptables rule viewer (FILTER + NAT tables) |
| 8 | **Setup** | Auto-detect interface, gateway, configure portal IP |

---

## Supported Distros

| Distro | Package Manager | Auto-Install |
|--------|----------------|-------------|
| Arch Linux / Manjaro | `pacman` | ✅ |
| Debian / Ubuntu / Kali | `apt` | ✅ |
| Fedora / RHEL / CentOS | `dnf` | ✅ |

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

> airjail auto-installs all missing dependencies on first run too.

---

## Manual Dependencies

**System packages:**
```bash
# Arch
sudo pacman -S python nmap aircrack-ng iptables iw net-tools

# Debian / Ubuntu / Kali
sudo apt install python3 python3-pip nmap aircrack-ng iptables iw net-tools libpcap-dev

# Fedora
sudo dnf install python3 python3-pip nmap aircrack-ng iptables iw net-tools libpcap-devel
```

**Python packages:**
```bash
pip install rich scapy requests manuf --break-system-packages
```

---

## Deauth Setup

Deauth requires a **second USB WiFi adapter** in monitor mode (do NOT use your main adapter):

```bash
sudo airmon-ng check kill
sudo airmon-ng start wlan1     # creates wlan1mon
```

Then in airjail → option `3` → enter `wlan1mon` as the monitor interface.

---

## Captive Portal / Jail Setup

1. Start a simple HTTP server on any LAN machine with your abuse warning page:
   ```bash
   python3 -m http.server 80
   ```
2. Create an `index.html` abuse/warning form in that folder
3. In airjail → option `6` → set portal IP → jail the abusive user

When jailed, the user's HTTP/HTTPS traffic is redirected to your form page via `iptables DNAT`.

---

## Usage Notes

- Must run as **root** — raw sockets + iptables require it
- DNS Spy works passively (no ARP poisoning needed) — it sniffs port 53 on your LAN interface
- Bandwidth monitor uses iptables FORWARD chain byte counters — resets when airjail exits
- Jailed users are tracked in-memory only — they auto-release if you restart

---

## Legal

> Only use airjail on networks **you own** or have **explicit permission** to manage.
> Unauthorized interception or disruption of network traffic is illegal in most countries.

---

## Author

**0xb0rn3 | oxbv1**
GitHub: [github.com/0xb0rn3/airjail](https://github.com/0xb0rn3/airjail)
