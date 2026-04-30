# 05 - Troubleshooting

## Purpose

This runbook covers common Tailscale + Mullvad exit-node failure modes and fixes.

## Troubleshooting Order

Use this order:

1. Check Tailscale status.
2. Check Tailscale version.
3. Confirm Mullvad add-on is active.
4. Confirm device has Mullvad access.
5. List available exit nodes.
6. Try a different Mullvad exit node.
7. Test public IP.
8. Test raw IP connectivity.
9. Test DNS.
10. Restart Tailscale.
11. Disable and re-enable exit node.

## Universal Status Commands

Linux/macOS:

```bash
tailscale status
```

```bash
tailscale version
```

```bash
tailscale exit-node list
```

Windows PowerShell:

```powershell
tailscale status
```

```powershell
tailscale version
```

```powershell
tailscale exit-node list
```

## Problem: Mullvad Exit Nodes Do Not Appear

### Symptoms

- `tailscale exit-node list` does not show Mullvad nodes.
- GUI does not show Mullvad locations.
- Only your own exit nodes appear.

### Likely Causes

- Mullvad add-on is not active.
- Device was not granted Mullvad access.
- You are logged into the wrong tailnet.
- Tailscale version is too old.
- Tailnet policy file does not include the `mullvad` node attribute.
- Tailnet Lock is enabled and Mullvad node keys are not signed.

### Fix

Check status:

```bash
tailscale status
```

Check version:

```bash
tailscale version
```

List exit nodes:

```bash
tailscale exit-node list
```

Confirm in the Tailscale admin console:

- Mullvad VPN add-on is active.
- The device has Mullvad access.
- The device is part of the correct tailnet.

Restart Tailscale.

Linux:

```bash
sudo systemctl restart tailscaled
```

macOS:

```bash
sudo tailscale down
```

```bash
sudo tailscale up
```

Windows PowerShell:

```powershell
Restart-Service Tailscale
```

List again:

```bash
tailscale exit-node list
```

## Problem: Internet Breaks After Selecting Mullvad Exit Node

### Symptoms

- Websites do not load.
- `ping 1.1.1.1` fails.
- `curl ifconfig.me` fails.
- Disabling exit node restores connectivity.

### Quick Fix

Disable exit node.

Linux/macOS:

```bash
sudo tailscale set --exit-node=
```

Windows PowerShell:

```powershell
tailscale set --exit-node=
```

Restart Tailscale.

Linux:

```bash
sudo systemctl restart tailscaled
```

macOS:

```bash
sudo tailscale down
```

```bash
sudo tailscale up
```

Windows PowerShell:

```powershell
Restart-Service Tailscale
```

Try another Mullvad exit node:

```bash
tailscale exit-node list
```

Linux/macOS:

```bash
sudo tailscale set --exit-node=another-mullvad-node.mullvad.ts.net --exit-node-allow-lan-access=true
```

Windows PowerShell:

```powershell
tailscale set --exit-node=another-mullvad-node.mullvad.ts.net --exit-node-allow-lan-access=true
```

## Problem: DNS Fails But IP Connectivity Works

### Symptoms

- `ping 1.1.1.1` works.
- `ping example.com` fails.
- Browsers fail to load websites.
- Public IP may be changed, but DNS resolution is broken.

### Confirm IP Connectivity

Linux/macOS:

```bash
ping -c 4 1.1.1.1
```

Windows PowerShell:

```powershell
ping 1.1.1.1
```

### Confirm DNS Failure

Linux/macOS:

```bash
ping -c 4 example.com
```

Windows PowerShell:

```powershell
ping example.com
```

## Linux DNS Fixes

Check DNS:

```bash
resolvectl status
```

Restart resolver:

```bash
sudo systemctl restart systemd-resolved
```

Restart Tailscale:

```bash
sudo systemctl restart tailscaled
```

Try not accepting Tailscale DNS:

```bash
sudo tailscale up --accept-dns=false
```

Re-enable Mullvad exit node:

```bash
sudo tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net --exit-node-allow-lan-access=true
```

## macOS DNS Fixes

Show DNS config:

```bash
scutil --dns
```

Flush DNS:

```bash
sudo dscacheutil -flushcache
```

Restart mDNSResponder:

```bash
sudo killall -HUP mDNSResponder
```

Restart Tailscale:

```bash
sudo tailscale down
```

```bash
sudo tailscale up
```

## Windows DNS Fixes

Show DNS servers:

```powershell
Get-DnsClientServerAddress
```

Flush DNS:

```powershell
ipconfig /flushdns
```

Restart Tailscale:

```powershell
Restart-Service Tailscale
```

## Problem: Local LAN Devices Are Not Reachable

### Symptoms

- Cannot reach local router.
- Cannot print.
- Cannot access NAS.
- Cannot access local dashboards.
- LAN worked before enabling Mullvad exit node.

### Cause

Local LAN access may be blocked unless explicitly allowed while using an exit node.

### Fix

Enable local LAN access.

Linux/macOS:

```bash
sudo tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net --exit-node-allow-lan-access=true
```

Windows PowerShell:

```powershell
tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net --exit-node-allow-lan-access=true
```

Verify routing.

Linux:

```bash
ip route
```

macOS:

```bash
netstat -rn
```

Windows PowerShell:

```powershell
route print
```

## Problem: Public IP Does Not Change

### Symptoms

- You selected a Mullvad exit node.
- Public IP still shows ISP/local IP.
- `curl ifconfig.me` does not match the Mullvad location.

### Checks

Check status:

```bash
tailscale status
```

List exit nodes:

```bash
tailscale exit-node list
```

Check public IP.

Linux/macOS:

```bash
curl -4 ifconfig.me && echo
```

Windows PowerShell:

```powershell
curl.exe -4 ifconfig.me
```

### Fix

Disable and re-enable exit node.

Linux/macOS:

```bash
sudo tailscale set --exit-node=
```

```bash
sudo tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net
```

Windows PowerShell:

```powershell
tailscale set --exit-node=
```

```powershell
tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net
```

Restart Tailscale if needed.

Linux:

```bash
sudo systemctl restart tailscaled
```

macOS:

```bash
sudo tailscale down
```

```bash
sudo tailscale up
```

Windows PowerShell:

```powershell
Restart-Service Tailscale
```

## Problem: Windows GUI Does Not Show All Mullvad Exit Nodes

### Cause

The Windows GUI may not display the full Mullvad exit-node list.

### Fix

Use PowerShell:

```powershell
tailscale exit-node list
```

Then select manually:

```powershell
tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net
```

## Problem: CLI Does Not Support "Best Available" Mullvad Exit Node

### Current Behavior

Some GUI clients may offer a "best available" Mullvad option.

The CLI may require a specific exit-node hostname or IP.

### Workaround

List nodes:

```bash
tailscale exit-node list
```

Pick a specific city or region.

Example:

```bash
sudo tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net
```

If performance is poor, try another city:

```bash
sudo tailscale set --exit-node=us-chi-wg-001.mullvad.ts.net
```

Disable exit node:

```bash
sudo tailscale set --exit-node=
```

On Windows PowerShell:

```powershell
tailscale set --exit-node=
```

## Problem: Tailnet Lock Blocks Mullvad Exit Nodes

### Symptoms

- Mullvad access is enabled.
- Device has Mullvad license access.
- Exit nodes appear but cannot be used.
- Tailnet Lock is enabled.

### Cause

If Tailnet Lock is enabled, Mullvad exit-node keys may need to be signed.

### Inspect Tailnet Lock

```bash
tailscale lock
```

### Sign a Mullvad Node Key

Run from a signing node:

```bash
tailscale lock sign nodekey:REPLACE_WITH_NODE_KEY
```

Then retry the Mullvad exit node.

## Problem: Another VPN Client Conflicts with Tailscale

### Symptoms

- Tailscale works until another VPN connects.
- Mullvad exit node fails when standalone VPN app is active.
- Routes change unexpectedly.
- DNS changes unexpectedly.
- Internet breaks after connecting multiple VPNs.

### Recommendation

Do not run the standalone Mullvad VPN app and Tailscale Mullvad exit nodes at the same time unless you intentionally want nested VPN behavior and understand the routing.

Check routes.

Linux:

```bash
ip route
```

macOS:

```bash
netstat -rn
```

Windows PowerShell:

```powershell
route print
```

Disable other VPN clients and retry Tailscale.

## Problem: Cannot Access Tailnet Devices While Using Mullvad Exit Node

### Checks

Check status:

```bash
tailscale status
```

Ping tailnet device:

```bash
tailscale ping DEVICE-NAME
```

Ping Tailscale IP:

```bash
ping 100.x.x.x
```

Ping MagicDNS name:

```bash
ping device-name
```

### Fixes

Restart Tailscale.

Linux:

```bash
sudo systemctl restart tailscaled
```

macOS:

```bash
sudo tailscale down
```

```bash
sudo tailscale up
```

Windows PowerShell:

```powershell
Restart-Service Tailscale
```

Confirm ACLs allow access between devices.

## Problem: Mullvad License Slots Are Exhausted

### Symptoms

- Some devices can use Mullvad exit nodes.
- New devices cannot see or use Mullvad exit nodes.
- Access is inconsistent across devices.

### Cause

Tailscale allocates Mullvad access by licensed device slots.

If all paid slots are already allocated, additional devices may not get Mullvad access.

### Fix

Remove unused devices from Mullvad access in the admin console.

Or remove access from the policy file.

Then reconnect the desired device.

Linux/macOS:

```bash
sudo tailscale down
```

```bash
sudo tailscale up
```

Windows PowerShell:

```powershell
tailscale down
```

```powershell
tailscale up
```

## Emergency Reset

Use this if connectivity is broken and you need to recover quickly.

Linux/macOS:

```bash
sudo tailscale set --exit-node=
```

```bash
sudo tailscale down
```

```bash
sudo tailscale up
```

Linux only:

```bash
sudo systemctl restart tailscaled
```

macOS DNS flush:

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
