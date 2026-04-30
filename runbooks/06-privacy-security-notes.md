# 06 - Privacy and Security Notes

## Purpose

This runbook explains the privacy and security model for using Tailscale with Mullvad VPN exit nodes.

## Privacy Model

Tailscale + Mullvad exit nodes provide VPN-style public internet egress through Mullvad infrastructure while preserving Tailscale private networking.

This can improve privacy from:

- Local network operators
- Public Wi-Fi networks
- ISPs
- Basic IP-based tracking
- Some regional content filtering

## What Tailscale Can See

Tailscale manages identity, devices, and tailnet membership.

Tailscale can generally know:

- Your Tailscale account identity
- Which devices are in your tailnet
- Which devices have Mullvad access
- Which devices select Mullvad exit nodes
- Administrative configuration and ACLs

## What Mullvad Can See

Mullvad receives egress traffic from the Tailscale-managed integration.

Mullvad does not receive your Tailscale account identity from Tailscale as part of normal use.

Mullvad may see:

- Traffic exiting through Mullvad infrastructure
- Destination IP metadata visible to VPN providers
- Timing metadata
- Traffic volume metadata

## What This Setup Does Not Provide

This setup does not make you anonymous from all parties.

It does not eliminate:

- Browser fingerprinting
- Logged-in account tracking
- Application telemetry
- Cookies
- Device fingerprinting
- Endpoint compromise
- Malware risk
- DNS/application leaks caused by misconfigured apps
- Tracking by services you authenticate into

## Standalone Mullvad App vs Tailscale Mullvad Exit Nodes

Standalone Mullvad app:

```text
Device -> Mullvad VPN app -> Mullvad server -> Internet
```

Tailscale Mullvad exit node:

```text
Device -> Tailscale -> Mullvad exit node -> Internet
```

## When Tailscale Mullvad Exit Nodes Are Better

The Tailscale integration is better when you want:

- Tailnet access
- Tailscale device identity
- Admin-managed device access
- Easier switching between private network and VPN egress
- Reduced dual-VPN client conflicts
- Centralized device management

## When Standalone Mullvad App May Be Better

The standalone Mullvad app may be better when you want:

- Direct Mullvad account usage
- Mullvad app-specific features
- VPN-only usage without Tailscale identity
- A setup independent from your Tailscale account
- More traditional consumer VPN behavior

## Do Not Run Multiple VPNs Unless Necessary

Avoid running the standalone Mullvad VPN client and Tailscale Mullvad exit nodes at the same time unless you know exactly how routing and DNS are being handled.

Multiple VPNs can cause:

- Broken DNS
- Broken routes
- No internet connectivity
- Incorrect public IP
- Split-tunnel conflicts
- Local LAN access issues

## Recommended Security Practices

Keep Tailscale updated:

```bash
tailscale version
```

Review Tailscale status:

```bash
tailscale status
```

List exit nodes before selecting:

```bash
tailscale exit-node list
```

Disable exit node when not needed on Linux/macOS:

```bash
sudo tailscale set --exit-node=
```

Disable exit node when not needed on Windows PowerShell:

```powershell
tailscale set --exit-node=
```

## Tailnet Security Recommendations

Recommended:

- Enable MFA on your identity provider.
- Remove old devices from the tailnet.
- Use device approval where appropriate.
- Review ACLs regularly.
- Use groups for policy clarity.
- Grant Mullvad access only to devices that need it.
- Consider Tailnet Lock for stronger node identity security.
- Avoid giving broad access to unmanaged devices.

## Local LAN Access Consideration

Allowing local LAN access is useful but less restrictive.

Enable LAN access when needed on Linux/macOS:

```bash
sudo tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net --exit-node-allow-lan-access=true
```

Enable LAN access when needed on Windows PowerShell:

```powershell
tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net --exit-node-allow-lan-access=true
```

Disable LAN access for stricter behavior on Linux/macOS:

```bash
sudo tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net --exit-node-allow-lan-access=false
```

Disable LAN access for stricter behavior on Windows PowerShell:

```powershell
tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net --exit-node-allow-lan-access=false
```

## DNS Security Notes

DNS behavior matters.

If DNS breaks, check:

Linux:

```bash
resolvectl status
```

macOS:

```bash
scutil --dns
```

Windows PowerShell:

```powershell
Get-DnsClientServerAddress
```

If DNS works incorrectly, test raw IP connectivity first.

Linux/macOS:

```bash
ping -c 4 1.1.1.1
```

Windows PowerShell:

```powershell
ping 1.1.1.1
```

Then test DNS.

Linux/macOS:

```bash
ping -c 4 example.com
```

Windows PowerShell:

```powershell
ping example.com
```

If IP works but hostname fails, the issue is probably DNS.

## Operational Safety Checklist

Before enabling a Mullvad exit node:

- Confirm Tailscale is updated.
- Confirm device has Mullvad access.
- Confirm you know how to disable the exit node.
- Confirm whether LAN access is needed.
- Confirm whether another VPN is running.
- Confirm baseline public IP.
- Confirm DNS works before changing routes.

Baseline public IP on Linux/macOS:

```bash
curl -4 ifconfig.me && echo
```

Baseline public IP on Windows PowerShell:

```powershell
curl.exe -4 ifconfig.me
```

After enabling exit node on Linux/macOS:

```bash
curl -4 ifconfig.me && echo
```

After enabling exit node on Windows PowerShell:

```powershell
curl.exe -4 ifconfig.me
```

## Incident Notes

If you lose connectivity after enabling a Mullvad exit node:

1. Disable the exit node.
2. Restart Tailscale.
3. Flush DNS.
4. Try a different exit node.
5. Check for another VPN client.
6. Check routing table.
7. Check DNS.
8. Reconnect Tailscale.

Linux disable:

```bash
sudo tailscale set --exit-node=
```

macOS disable:

```bash
sudo tailscale set --exit-node=
```

Windows disable:

```powershell
tailscale set --exit-node=
```

## Linux Incident Commands

Disable exit node:

```bash
sudo tailscale set --exit-node=
```

Restart Tailscale:

```bash
sudo systemctl restart tailscaled
```

Check routes:

```bash
ip route
```

Check DNS:

```bash
resolvectl status
```

Check public IP:

```bash
curl -4 ifconfig.me && echo
```

## macOS Incident Commands

Disable exit node:

```bash
sudo tailscale set --exit-node=
```

Restart Tailscale:

```bash
sudo tailscale down
```

```bash
sudo tailscale up
```

Flush DNS:

```bash
sudo dscacheutil -flushcache
```

```bash
sudo killall -HUP mDNSResponder
```

Check routes:

```bash
netstat -rn
```

Check DNS:

```bash
scutil --dns
```

Check public IP:

```bash
curl -4 ifconfig.me && echo
```

## Windows Incident Commands

Disable exit node:

```powershell
tailscale set --exit-node=
```

Restart Tailscale:

```powershell
Restart-Service Tailscale
```

Flush DNS:

```powershell
ipconfig /flushdns
```

Check routes:

```powershell
route print
```

Check DNS:

```powershell
Get-DnsClientServerAddress
```

Check public IP:

```powershell
curl.exe -4 ifconfig.me
```

## Final Security Notes

This setup is strong for everyday privacy, safer networking, remote work, and operational convenience.

It is not a complete anonymity system.

For best results:

- Keep devices patched.
- Keep browsers hardened.
- Use MFA.
- Avoid unnecessary browser extensions.
- Do not mix VPN clients unless needed.
- Review tailnet devices regularly.
- Remove stale devices.
- Review access policies regularly.
