#!/usr/bin/env bash
# ─────────────────────────────────────────────────────────────────────────────
#  airjail — cross-distro installer
#  github: https://github.com/0xb0rn3/airjail
#  dev: 0xb0rn3 | oxbv1
# ─────────────────────────────────────────────────────────────────────────────

set -e

RED='\033[0;31m'
GREEN='\033[0;32m'
CYAN='\033[0;36m'
YELLOW='\033[1;33m'
BOLD='\033[1m'
NC='\033[0m'

print_banner() {
cat << 'EOF'

  █████╗ ██╗██████╗      ██╗ █████╗ ██╗██╗
 ██╔══██╗██║██╔══██╗     ██║██╔══██╗██║██║
 ███████║██║██████╔╝     ██║███████║██║██║
 ██╔══██║██║██╔══██╗██   ██║██╔══██║██║██║
 ██║  ██║██║██║  ██║╚█████╔╝██║  ██║██║███████╗
 ╚═╝  ╚═╝╚═╝╚═╝  ╚═╝ ╚════╝ ╚═╝  ╚═╝╚═╝╚══════╝

  WiFi Network Manager & Abuse Control
  dev: 0xb0rn3 | oxbv1
  https://github.com/0xb0rn3/airjail

EOF
}

ok()   { echo -e "  ${GREEN}✓${NC} $1"; }
info() { echo -e "  ${CYAN}*${NC} $1"; }
warn() { echo -e "  ${YELLOW}!${NC} $1"; }
fail() { echo -e "  ${RED}✗${NC} $1"; exit 1; }

# ── root check ─────────────────────────────────────────────────────────────
[[ $EUID -ne 0 ]] && fail "Run as root: sudo ./install"

print_banner

# ── detect distro ──────────────────────────────────────────────────────────
detect_pm() {
    if   command -v pacman &>/dev/null; then echo "pacman"
    elif command -v apt    &>/dev/null; then echo "apt"
    elif command -v dnf    &>/dev/null; then echo "dnf"
    else echo "unknown"
    fi
}

PM=$(detect_pm)
info "Detected package manager: ${BOLD}$PM${NC}"

DISTRO=$(grep -oP '(?<=^NAME=).*' /etc/os-release 2>/dev/null | tr -d '"' || echo "Unknown")
info "Distro: $DISTRO"

# ── install system packages ────────────────────────────────────────────────
install_sys() {
    case "$PM" in
        pacman)
            info "Updating pacman..."
            pacman -Sy --noconfirm 2>/dev/null
            info "Installing system packages..."
            pacman -S --noconfirm --needed \
                python python-pip python-scapy \
                nmap net-tools aircrack-ng \
                iptables iw wireless_tools \
                2>/dev/null && ok "System packages installed"
            ;;
        apt)
            info "Updating apt..."
            apt update -qq 2>/dev/null
            info "Installing system packages..."
            DEBIAN_FRONTEND=noninteractive apt install -y -qq \
                python3 python3-pip python3-scapy \
                nmap net-tools aircrack-ng \
                iptables iw wireless-tools \
                libpcap-dev 2>/dev/null && ok "System packages installed"
            ;;
        dnf)
            info "Installing system packages..."
            dnf install -y -q \
                python3 python3-pip \
                nmap net-tools aircrack-ng \
                iptables iw wireless-tools \
                libpcap-devel 2>/dev/null && ok "System packages installed"
            ;;
        *)
            warn "Unknown package manager. Install manually:"
            echo "  Arch  : sudo pacman -S python nmap aircrack-ng iptables iw"
            echo "  Debian: sudo apt install python3 nmap aircrack-ng iptables iw"
            echo "  Fedora: sudo dnf install python3 nmap aircrack-ng iptables iw"
            ;;
    esac
}

# ── install python packages ────────────────────────────────────────────────
install_py() {
    info "Installing Python packages..."
    PY_FLAGS=""
    [[ "$PM" == "apt" || "$PM" == "dnf" ]] && PY_FLAGS="--break-system-packages"
    
    for pkg in rich scapy requests manuf; do
        python3 -m pip install "$pkg" -q $PY_FLAGS 2>/dev/null \
            && ok "pip: $pkg" \
            || warn "Could not install $pkg — try: pip install $pkg $PY_FLAGS"
    done
}

# ── enable ip forwarding ────────────────────────────────────────────────────
enable_ipfwd() {
    sysctl -w net.ipv4.ip_forward=1 > /dev/null
    grep -q "net.ipv4.ip_forward" /etc/sysctl.conf \
        && sed -i 's/.*net.ipv4.ip_forward.*/net.ipv4.ip_forward=1/' /etc/sysctl.conf \
        || echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
    ok "IP forwarding enabled"
}

# ── create launcher ─────────────────────────────────────────────────────────
create_launcher() {
    SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
    cat > /usr/local/bin/airjail << EOF
#!/usr/bin/env bash
sudo python3 $SCRIPT_DIR/airjail"\$@"
EOF
    chmod +x /usr/local/bin/airjail
    ok "Launcher created → run: sudo airjail"
}

# ── run steps ───────────────────────────────────────────────────────────────
install_sys
install_py
enable_ipfwd
create_launcher

echo ""
echo -e "${GREEN}${BOLD}  ✓ airjail installed successfully!${NC}"
echo ""
echo -e "  Run with: ${CYAN}sudo airjail${NC}"
echo -e "  Or      : ${CYAN}sudo python3 airjail${NC}"
echo ""
echo -e "  ${YELLOW}Tip: For deauth, plug in a second USB WiFi adapter then:${NC}"
echo -e "  ${CYAN}sudo airmon-ng check kill${NC}"
echo -e "  ${CYAN}sudo airmon-ng start wlan1${NC}"
echo ""
