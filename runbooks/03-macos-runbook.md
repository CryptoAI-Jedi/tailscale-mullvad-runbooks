# 03 - macOS Runbook

## Purpose

This runbook covers Tailscale + Mullvad VPN exit-node setup and usage on macOS.

## Requirements

- macOS device
- Tailscale account
- Tailscale installed
- Tailscale client version `1.48.2` or newer
- Mullvad VPN add-on enabled through Tailscale
- Device granted Mullvad access
- Admin privileges

## Install Tailscale on macOS

Option 1: install Tailscale from the official download page.

Option 2: install with Homebrew:

```bash
brew install --cask tailscale
```

Open Tailscale:

```bash
open -a Tailscale
```

Authenticate through the browser.

## Verify Tailscale Version

```bash
tailscale version
```

If the CLI is not available, try:

```bash
/Applications/Tailscale.app/Contents/MacOS/Tailscale version
```

Recommended:

```text
Tailscale v1.48.3 or later
```

## Check Tailscale Status

```bash
tailscale status
```

If the CLI is not available:

```bash
/Applications/Tailscale.app/Contents/MacOS/Tailscale status
```

## List Available Mullvad Exit Nodes

```bash
tailscale exit-node list
```

If the CLI is not available:

```bash
/Applications/Tailscale.app/Contents/MacOS/Tailscale exit-node list
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

Replace this hostname with a real hostname from:

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

Use this if you need access to:

- Local router
- Printer
- NAS
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

## macOS GUI Method

1. Open the Tailscale menu bar icon.
2. Go to **Exit Node**.
3. Select a Mullvad location.
4. Enable or disable local network access depending on your needs.
5. Verify public IP changed.

## Verify Public IP

```bash
curl -4 ifconfig.me && echo
```

Check geolocation:

```bash
curl ipinfo.io
```

Check IPv6:

```bash
curl -6 ifconfig.me && echo
```

If IPv6 fails, it does not always mean the exit node is broken.

## Verify Tailnet Access

Check Tailscale status:

```bash
tailscale status
```

Ping a tailnet device:

```bash
tailscale ping DEVICE-NAME
```

Ping Tailscale IP:

```bash
ping -c 4 100.x.x.x
```

Ping MagicDNS name:

```bash
ping -c 4 device-name
```

## Verify Routing

Show routing table:

```bash
netstat -rn
```

Show default route:

```bash
route get default
```

Show route to a public IP:

```bash
route get 1.1.1.1
```

## Verify DNS

Show DNS configuration:

```bash
scutil --dns
```

Check network services:

```bash
networksetup -listallnetworkservices
```

Check network service order:

```bash
networksetup -listnetworkserviceorder
```

Test DNS:

```bash
dig example.com
```

If `dig` is unavailable, use:

```bash
nslookup example.com
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

## Flush DNS Cache

```bash
sudo dscacheutil -flushcache
```

Restart mDNSResponder:

```bash
sudo killall -HUP mDNSResponder
```

## Bring Tailscale Down and Up

```bash
sudo tailscale down
```

```bash
sudo tailscale up
```

## Full macOS Reset

Disable exit node:

```bash
sudo tailscale set --exit-node=
```

Bring Tailscale down:

```bash
sudo tailscale down
```

Bring Tailscale up:

```bash
sudo tailscale up
```

Flush DNS:

```bash
sudo dscacheutil -flushcache
```

Restart mDNSResponder:

```bash
sudo killall -HUP mDNSResponder
```

Check public IP:

```bash
curl -4 ifconfig.me && echo
```

Check status:

```bash
tailscale status
```

## Useful macOS Aliases

Add to Zsh config:

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
alias ts-routes='netstat -rn'
alias ts-dns='scutil --dns'
```

Reload:

```bash
source ~/.zshrc
```

## macOS Function: Quickly Switch Mullvad Exit Node

Add to `~/.zshrc`:

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

Reload:

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

## Quick macOS Health Check

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
netstat -rn
```

```bash
scutil --dns
```
