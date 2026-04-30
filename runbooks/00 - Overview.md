# 00 - Overview

## Purpose

This runbook explains the operating model for using **Tailscale with Mullvad VPN exit nodes**.

The goal is to route public internet traffic through Mullvad VPN infrastructure while preserving normal Tailscale private-network access.

## What This Setup Is

This is the official Tailscale + Mullvad integration.

Mullvad VPN servers appear as Tailscale exit nodes.

Your device connects to Tailscale, selects a Mullvad exit node, and sends internet-bound traffic through that selected exit node.

## What This Setup Is Not

This is not the same as:

- Running the standalone Mullvad VPN client
- Running two independent VPN clients at the same time
- Importing a separate Mullvad account into Tailscale
- Manually configuring WireGuard profiles
- Hosting your own exit node
- Running Mullvad as a subnet router

## Basic Traffic Flow

```text
Device
  |
  | Tailscale tunnel
  v
Tailscale tailnet
  |
  | Selected Mullvad exit node
  v
Mullvad VPN infrastructure
  |
  v
Public Internet
```

## Normal Tailscale Behavior

Without a Mullvad exit node:

```text
Device -> Tailscale tailnet -> private devices/services
```

Public internet traffic exits normally through your local ISP or current network.

## Tailscale with Mullvad Exit Node Behavior

With a Mullvad exit node enabled:

```text
Device -> Tailscale tailnet -> Mullvad exit node -> public internet
```

Private tailnet traffic should continue working through Tailscale.

Public internet traffic should egress through Mullvad.

## Why Use This Setup

Use this setup if you want:

- VPN-style internet egress
- Tailscale private-network access
- Easier device-level VPN management
- Reduced dual-VPN routing conflicts
- Exit-node switching from the Tailscale client
- Mullvad egress without running the standalone Mullvad app

## Common Use Cases

- Safer public Wi-Fi usage
- Remote work privacy
- VPN-style browsing
- Testing websites from different regions
- Keeping tailnet access while using VPN egress
- Simplifying Tailscale + VPN compatibility

## Requirements

- Tailscale account
- Tailscale client installed
- Tailscale client version `1.48.2` or newer
- Mullvad VPN add-on purchased through Tailscale
- Device granted Mullvad access
- Admin/root privileges on the device

## Recommended Version

Check Tailscale version:

```bash
tailscale version
```

Recommended:

```text
Tailscale v1.48.3 or later
```

Use the newest stable Tailscale client when possible.

## Important Limitations

- Mullvad exit nodes require the Mullvad add-on through Tailscale.
- A separate Mullvad account number is not used directly for this integration.
- Mullvad exit nodes do not work with custom DERP servers.
- Windows may not show the full Mullvad exit-node list in the GUI.
- The CLI is often the best way to list and select Mullvad exit nodes.
- Tailnet Lock may require Mullvad node keys to be signed.
- Mullvad access should be managed through either the admin console or policy file, not both.
- Older Tailscale versions may have DNS issues with Mullvad exit nodes.

## Key Commands

List available exit nodes:

```bash
tailscale exit-node list
```

Set an exit node on Linux/macOS:

```bash
sudo tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net
```

Set an exit node on Windows PowerShell:

```powershell
tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net
```

Set an exit node with LAN access on Linux/macOS:

```bash
sudo tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net --exit-node-allow-lan-access=true
```

Set an exit node with LAN access on Windows PowerShell:

```powershell
tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net --exit-node-allow-lan-access=true
```

Disable exit node on Linux/macOS:

```bash
sudo tailscale set --exit-node=
```

Disable exit node on Windows PowerShell:

```powershell
tailscale set --exit-node=
```

Check public IP:

```bash
curl -4 ifconfig.me && echo
```

## When to Enable LAN Access

Enable LAN access if you need to reach:

- Local router
- Printer
- NAS
- Home lab dashboard
- Local network services
- Local IoT devices

Linux/macOS:

```bash
sudo tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net --exit-node-allow-lan-access=true
```

Windows PowerShell:

```powershell
tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net --exit-node-allow-lan-access=true
```

## When to Disable LAN Access

Disable LAN access if you want stricter isolation from the local network.

Linux/macOS:

```bash
sudo tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net --exit-node-allow-lan-access=false
```

Windows PowerShell:

```powershell
tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net --exit-node-allow-lan-access=false
```

## Recommended Troubleshooting Order

1. Confirm Tailscale is running.
2. Confirm Tailscale version.
3. Confirm Mullvad add-on is active.
4. Confirm the device has Mullvad access.
5. List available exit nodes.
6. Select a different Mullvad exit node.
7. Test public IP.
8. Test raw IP connectivity.
9. Test DNS.
10. Restart Tailscale.
11. Disable and re-enable the exit node.

## Baseline Verification Before Enabling Exit Node

Check Tailscale:

```bash
tailscale status
```

Check public IP:

```bash
curl -4 ifconfig.me && echo
```

Check DNS:

```bash
ping -c 4 example.com
```

Check internet by IP:

```bash
ping -c 4 1.1.1.1
```

On Windows PowerShell:

```powershell
tailscale status
```

```powershell
curl.exe -4 ifconfig.me
```

```powershell
ping example.com
```

```powershell
ping 1.1.1.1
```

## Verification After Enabling Exit Node

Check public IP:

```bash
curl -4 ifconfig.me && echo
```

Check geolocation:

```bash
curl ipinfo.io
```

Check Tailscale status:

```bash
tailscale status
```

Check exit nodes:

```bash
tailscale exit-node list
```

Test DNS:

```bash
ping -c 4 example.com
```

Test internet by IP:

```bash
ping -c 4 1.1.1.1
```

On Windows PowerShell:

```powershell
curl.exe -4 ifconfig.me
```

```powershell
curl.exe ipinfo.io
```

```powershell
tailscale status
```

```powershell
tailscale exit-node list
```

```powershell
ping example.com
```

```powershell
ping 1.1.1.1
```
