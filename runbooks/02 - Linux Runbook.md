# 02 - Linux Runbook

## Purpose

This runbook covers Tailscale + Mullvad VPN exit-node setup and usage on Linux.

Applies to:

- Ubuntu
- Debian
- Arch Linux
- Fedora
- CentOS
- RHEL
- Most systemd-based Linux distributions

## Requirements

- Tailscale account
- Tailscale installed
- Tailscale client version `1.48.2` or newer
- Mullvad VPN add-on enabled through Tailscale
- Device granted Mullvad access
- Root/sudo access

## Install Tailscale on Ubuntu/Debian

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Start and enable Tailscale:

```bash
sudo systemctl enable --now tailscaled
```

Authenticate device:

```bash
sudo tailscale up
```

Check status:

```bash
tailscale status
```

## Install Tailscale on Arch Linux

```bash
sudo pacman -Syu tailscale
```

Start and enable Tailscale:

```bash
sudo systemctl enable --now tailscaled
```

Authenticate device:

```bash
sudo tailscale up
```

Check status:

```bash
tailscale status
```

## Install Tailscale on Fedora

```bash
sudo dnf install tailscale -y
```

Start and enable Tailscale:

```bash
sudo systemctl enable --now tailscaled
```

Authenticate device:

```bash
sudo tailscale up
```

Check status:

```bash
tailscale status
```

## Install Tailscale on CentOS/RHEL

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Start and enable Tailscale:

```bash
sudo systemctl enable --now tailscaled
```

Authenticate device:

```bash
sudo tailscale up
```

Check status:

```bash
tailscale status
```

## Verify Tailscale Version

```bash
tailscale version
```

Recommended:

```text
Tailscale v1.48.3 or later
```

## Confirm Tailscale Service Status

```bash
systemctl status tailscaled
```

If not running:

```bash
sudo systemctl start tailscaled
```

Enable at boot:

```bash
sudo systemctl enable tailscaled
```

## Confirm Tailnet Status

```bash
tailscale status
```

Look for:

- Your device is online
- Other tailnet devices appear
- No authentication errors
- No expired node key warning

## List Available Mullvad Exit Nodes

```bash
tailscale exit-node list
```

Expected output may include hostnames like:

```text
us-nyc-wg-001.mullvad.ts.net
gb-lon-wg-001.mullvad.ts.net
nl-ams-wg-001.mullvad.ts.net
```

If no Mullvad nodes appear, check:

- Mullvad add-on is active.
- Device has Mullvad access.
- Tailscale client is updated.
- Device is logged into the correct tailnet.

## Use a Mullvad Exit Node

Replace the hostname with a real hostname from:

```bash
tailscale exit-node list
```

Set exit node:

```bash
sudo tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net
```

## Use Mullvad Exit Node with LAN Access

```bash
sudo tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net --exit-node-allow-lan-access=true
```

Use this if you need access to local devices such as:

- Printer
- NAS
- Router
- Local dashboard
- Home lab machine

## Disable Local LAN Access

```bash
sudo tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net --exit-node-allow-lan-access=false
```

## Disable Mullvad Exit Node

```bash
sudo tailscale set --exit-node=
```

## Verify Public IP

```bash
curl -4 ifconfig.me && echo
```

Check IP geolocation:

```bash
curl ipinfo.io
```

Check IPv6:

```bash
curl -6 ifconfig.me && echo
```

If IPv6 fails, that does not always mean the exit node is broken. Some networks or exit paths may not provide IPv6.

## Verify Tailnet Access

Check status:

```bash
tailscale status
```

Ping a tailnet device:

```bash
tailscale ping DEVICE-NAME
```

Ping a Tailscale IP:

```bash
ping -c 4 100.x.x.x
```

Ping a MagicDNS name:

```bash
ping -c 4 device-name
```

## Verify Routing

Show routing table:

```bash
ip route
```

Show default route:

```bash
ip route show default
```

Show Tailscale interface:

```bash
ip addr show tailscale0
```

Show all interfaces:

```bash
ip addr
```

## Verify DNS

Check resolver status:

```bash
resolvectl status
```

Test DNS resolution:

```bash
getent hosts example.com
```

If `dig` is installed:

```bash
dig example.com
```

If `dig` is missing on Ubuntu/Debian:

```bash
sudo apt update
```

```bash
sudo apt install dnsutils -y
```

If `dig` is missing on Arch:

```bash
sudo pacman -S bind
```

If `dig` is missing on Fedora/RHEL/CentOS:

```bash
sudo dnf install bind-utils -y
```

## Test Internet by IP

```bash
ping -c 4 1.1.1.1
```

## Test Internet by DNS

```bash
ping -c 4 example.com
```

## Test HTTPS

```bash
curl -I https://example.com
```

## Restart Tailscale

```bash
sudo systemctl restart tailscaled
```

Check service:

```bash
systemctl status tailscaled
```

## Bring Tailscale Down and Up

```bash
sudo tailscale down
```

```bash
sudo tailscale up
```

## Run Tailscale Without Accepting Tailscale DNS

Use this if DNS breaks when the exit node is enabled.

```bash
sudo tailscale up --accept-dns=false
```

Then enable Mullvad exit node:

```bash
sudo tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net --exit-node-allow-lan-access=true
```

## Revert to Accepting Tailscale DNS

```bash
sudo tailscale up --accept-dns=true
```

## Full Linux Reset

Disable exit node:

```bash
sudo tailscale set --exit-node=
```

Restart Tailscale:

```bash
sudo systemctl restart tailscaled
```

Bring Tailscale down:

```bash
sudo tailscale down
```

Bring Tailscale up:

```bash
sudo tailscale up
```

Check status:

```bash
tailscale status
```

Check public IP:

```bash
curl -4 ifconfig.me && echo
```

## Useful Linux Aliases

Add to Bash config:

```bash
nano ~/.bashrc
```

Or Zsh config:

```bash
nano ~/.zshrc
```

Add:

```bash
alias ts-status='tailscale status'
alias ts-version='tailscale version'
alias ts-mullvad-list='tailscale exit-node list'
alias ts-mullvad-off='sudo tailscale set --exit-node='
alias ts-ip='curl -4 ifconfig.me && echo'
alias ts-routes='ip route'
alias ts-dns='resolvectl status'
```

Reload Bash:

```bash
source ~/.bashrc
```

Reload Zsh:

```bash
source ~/.zshrc
```

## Linux Function: Quickly Switch Mullvad Exit Node

Add to `~/.bashrc` or `~/.zshrc`:

```bash
ts-mullvad-use() {
  if [ -z "\$1" ]; then
    echo "Usage: ts-mullvad-use <mullvad-exit-node-hostname>"
    echo "Example: ts-mullvad-use us-nyc-wg-001.mullvad.ts.net"
    return 1
  fi

  sudo tailscale set \
    --exit-node="\$1" \
    --exit-node-allow-lan-access=true
}
```

Reload shell config:

```bash
source ~/.bashrc
```

Or:

```bash
source ~/.zshrc
```

Use:

```bash
ts-mullvad-use us-nyc-wg-001.mullvad.ts.net
```

Disable:

```bash
ts-mullvad-off
```

## Quick Linux Health Check

Run after enabling a Mullvad exit node:

```bash
tailscale status
```

```bash
curl -4 ifconfig.me && echo
```

```bash
curl ipinfo.io
```

```bash
ping -c 4 1.1.1.1
```

```bash
ping -c 4 example.com
```

```bash
ip route
```

```bash
resolvectl status
```
