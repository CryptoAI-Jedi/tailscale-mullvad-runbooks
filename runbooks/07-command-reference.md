# 07 - Command Reference

## Purpose

Quick command reference for Tailscale + Mullvad VPN exit-node usage.

## Universal Commands

Check status:

```bash
tailscale status
```

Check version:

```bash
tailscale version
```

List available exit nodes:

```bash
tailscale exit-node list
```

## Linux/macOS Commands

Set Mullvad exit node:

```bash
sudo tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net
```

Set Mullvad exit node with LAN access:

```bash
sudo tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net --exit-node-allow-lan-access=true
```

Set Mullvad exit node without LAN access:

```bash
sudo tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net --exit-node-allow-lan-access=false
```

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

Check public IPv4:

```bash
curl -4 ifconfig.me && echo
```

Check public IPv6:

```bash
curl -6 ifconfig.me && echo
```

Check IP geolocation:

```bash
curl ipinfo.io
```

Test internet by IP:

```bash
ping -c 4 1.1.1.1
```

Test DNS:

```bash
ping -c 4 example.com
```

Test HTTPS:

```bash
curl -I https://example.com
```

## Linux-Specific Commands

Restart Tailscale:

```bash
sudo systemctl restart tailscaled
```

Check Tailscale service:

```bash
systemctl status tailscaled
```

Enable Tailscale at boot:

```bash
sudo systemctl enable tailscaled
```

Start Tailscale:

```bash
sudo systemctl start tailscaled
```

Check routes:

```bash
ip route
```

Check default route:

```bash
ip route show default
```

Check Tailscale interface:

```bash
ip addr show tailscale0
```

Check all interfaces:

```bash
ip addr
```

Check DNS:

```bash
resolvectl status
```

Restart systemd-resolved:

```bash
sudo systemctl restart systemd-resolved
```

Use Tailscale without accepting Tailscale DNS:

```bash
sudo tailscale up --accept-dns=false
```

Re-enable Tailscale DNS:

```bash
sudo tailscale up --accept-dns=true
```

## macOS-Specific Commands

Open Tailscale:

```bash
open -a Tailscale
```

Check routes:

```bash
netstat -rn
```

Check default route:

```bash
route get default
```

Check route to public IP:

```bash
route get 1.1.1.1
```

Check DNS:

```bash
scutil --dns
```

List network services:

```bash
networksetup -listallnetworkservices
```

List network service order:

```bash
networksetup -listnetworkserviceorder
```

Flush DNS:

```bash
sudo dscacheutil -flushcache
```

Restart mDNSResponder:

```bash
sudo killall -HUP mDNSResponder
```

## Windows PowerShell Commands

Check status:

```powershell
tailscale status
```

Check version:

```powershell
tailscale version
```

List exit nodes:

```powershell
tailscale exit-node list
```

Set Mullvad exit node:

```powershell
tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net
```

Set Mullvad exit node with LAN access:

```powershell
tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net --exit-node-allow-lan-access=true
```

Set Mullvad exit node without LAN access:

```powershell
tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net --exit-node-allow-lan-access=false
```

Disable exit node:

```powershell
tailscale set --exit-node=
```

Bring Tailscale down:

```powershell
tailscale down
```

Bring Tailscale up:

```powershell
tailscale up
```

Restart Tailscale service:

```powershell
Restart-Service Tailscale
```

Check public IPv4:

```powershell
curl.exe -4 ifconfig.me
```

Check IP geolocation:

```powershell
curl.exe ipinfo.io
```

Check IP config:

```powershell
ipconfig /all
```

Check routes:

```powershell
route print
```

Check DNS servers:

```powershell
Get-DnsClientServerAddress
```

Flush DNS:

```powershell
ipconfig /flushdns
```

Test internet by IP:

```powershell
ping 1.1.1.1
```

Test DNS:

```powershell
ping example.com
```

Test HTTPS:

```powershell
curl.exe -I https://example.com
```

## Windows CLI Path

If `tailscale` is not recognized:

```powershell
& "C:\Program Files\Tailscale\tailscale.exe" version
```

Example:

```powershell
& "C:\Program Files\Tailscale\tailscale.exe" exit-node list
```

## Linux/macOS Alias Pack

Add to `~/.bashrc` or `~/.zshrc`:

```bash
alias ts-status='tailscale status'
alias ts-version='tailscale version'
alias ts-mullvad-list='tailscale exit-node list'
alias ts-mullvad-off='sudo tailscale set --exit-node='
alias ts-ip='curl -4 ifconfig.me && echo'
alias ts-geo='curl ipinfo.io'
```

Reload Bash:

```bash
source ~/.bashrc
```

Reload Zsh:

```bash
source ~/.zshrc
```

## Linux/macOS Function

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

Reload Bash:

```bash
source ~/.bashrc
```

Reload Zsh:

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

## Windows PowerShell Function

Open PowerShell profile:

```powershell
notepad $PROFILE
```

Create profile if needed:

```powershell
New-Item -ItemType File -Path $PROFILE -Force
```

Add:

```powershell
function Use-TailscaleMullvad {
    param (
        [Parameter(Mandatory=$true)]
        [string]$ExitNode
    )

    tailscale set --exit-node=$ExitNode --exit-node-allow-lan-access=true
}

function Disable-TailscaleMullvad {
    tailscale set --exit-node=
}
```

Reload profile:

```powershell
. $PROFILE
```

Use:

```powershell
Use-TailscaleMullvad us-nyc-wg-001.mullvad.ts.net
```

Disable:

```powershell
Disable-TailscaleMullvad
```

## Emergency Disable Commands

Linux/macOS:

```bash
sudo tailscale set --exit-node=
```

Windows PowerShell:

```powershell
tailscale set --exit-node=
```

## Emergency Reset Commands

Linux:

```bash
sudo tailscale set --exit-node=
```

```bash
sudo systemctl restart tailscaled
```

```bash
sudo tailscale down
```

```bash
sudo tailscale up
```

macOS:

```bash
sudo tailscale set --exit-node=
```

```bash
sudo tailscale down
```

```bash
sudo tailscale up
```

```bash
sudo dscacheutil -flushcache
```

```bash
sudo killall -HUP mDNSResponder
```

Windows PowerShell:

```powershell
tailscale set --exit-node=
```

```powershell
tailscale down
```

```powershell
tailscale up
```

```powershell
ipconfig /flushdns
```

```powershell
Restart-Service Tailscale
```
