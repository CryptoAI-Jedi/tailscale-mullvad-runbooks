# 04 - Windows Runbook

## Purpose

This runbook covers Tailscale + Mullvad VPN exit-node setup and usage on Windows.

Windows users can use:

- Tailscale GUI
- PowerShell
- `tailscale.exe` CLI

Important: the Windows Tailscale GUI may not display the full Mullvad exit-node list. Use the CLI if needed.

## Requirements

- Windows machine
- Tailscale account
- Tailscale installed
- Tailscale client version `1.48.2` or newer
- Mullvad VPN add-on enabled through Tailscale
- Device granted Mullvad access
- Administrator privileges

## Install Tailscale on Windows

Option 1: install from the official Tailscale download page.

Option 2: install with Winget:

```powershell
winget install tailscale.tailscale
```

Restart PowerShell after installation.

Authenticate through the Tailscale GUI.

## Open PowerShell as Administrator

1. Press `Start`.
2. Search `PowerShell`.
3. Right-click **Windows PowerShell**.
4. Select **Run as administrator**.

## Find Tailscale CLI

Try:

```powershell
tailscale version
```

If not found, try:

```powershell
& "C:\Program Files\Tailscale\tailscale.exe" version
```

For the rest of this runbook, if `tailscale` does not work, replace it with:

```powershell
& "C:\Program Files\Tailscale\tailscale.exe"
```

Example:

```powershell
& "C:\Program Files\Tailscale\tailscale.exe" status
```

## Verify Tailscale Version

```powershell
tailscale version
```

Recommended:

```text
Tailscale v1.48.3 or later
```

## Check Tailscale Status

```powershell
tailscale status
```

Look for:

- Your device is online
- Correct tailnet
- Other tailnet devices appear
- No authentication errors
- No expired node key warning

## List Available Mullvad Exit Nodes

```powershell
tailscale exit-node list
```

Expected output may include hostnames like:

```text
us-nyc-wg-001.mullvad.ts.net
gb-lon-wg-001.mullvad.ts.net
nl-ams-wg-001.mullvad.ts.net
```

If the GUI does not show all Mullvad locations, rely on the CLI output.

## Use a Mullvad Exit Node

Replace this hostname with a real hostname from:

```powershell
tailscale exit-node list
```

Set exit node:

```powershell
tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net
```

## Use Mullvad Exit Node with LAN Access

```powershell
tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net --exit-node-allow-lan-access=true
```

Use this if you need access to:

- Local router
- Printer
- NAS
- Local dashboard
- Home lab machine

## Disable Local LAN Access

```powershell
tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net --exit-node-allow-lan-access=false
```

## Disable Mullvad Exit Node

```powershell
tailscale set --exit-node=
```

## Windows GUI Method

1. Click the Tailscale tray icon.
2. Open **Exit Node**.
3. Select a Mullvad location.
4. Enable or disable local network access depending on your needs.
5. Verify your public IP.

## Verify Public IP

```powershell
curl.exe -4 ifconfig.me
```

Check geolocation:

```powershell
curl.exe ipinfo.io
```

## Verify Tailnet Access

Check status:

```powershell
tailscale status
```

Ping a tailnet device:

```powershell
tailscale ping DEVICE-NAME
```

Ping Tailscale IP:

```powershell
ping 100.x.x.x
```

Ping MagicDNS name:

```powershell
ping device-name
```

## Verify Windows IP Configuration

```powershell
ipconfig /all
```

## Verify Routing

```powershell
route print
```

## Verify DNS Client Servers

```powershell
Get-DnsClientServerAddress
```

## Test Internet by IP

```powershell
ping 1.1.1.1
```

## Test Internet by DNS

```powershell
ping example.com
```

## Test HTTPS

```powershell
curl.exe -I https://example.com
```

## Flush DNS Cache

```powershell
ipconfig /flushdns
```

## Restart Tailscale Service

```powershell
Restart-Service Tailscale
```

## Bring Tailscale Down and Up

```powershell
tailscale down
```

```powershell
tailscale up
```

## Full Windows Reset

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

Flush DNS:

```powershell
ipconfig /flushdns
```

Restart Tailscale service:

```powershell
Restart-Service Tailscale
```

Check public IP:

```powershell
curl.exe -4 ifconfig.me
```

Check status:

```powershell
tailscale status
```

## PowerShell Helper Function

Add this to your PowerShell profile.

Open profile:

```powershell
notepad $PROFILE
```

If the profile file does not exist, create it:

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
```

Reload profile:

```powershell
. $PROFILE
```

Use:

```powershell
Use-TailscaleMullvad us-nyc-wg-001.mullvad.ts.net
```

## PowerShell Helper: Disable Mullvad Exit Node

Add this to your PowerShell profile:

```powershell
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
Disable-TailscaleMullvad
```

## Quick Windows Health Check

Run after enabling a Mullvad exit node:

```powershell
tailscale status
```

```powershell
curl.exe -4 ifconfig.me
```

```powershell
curl.exe ipinfo.io
```

```powershell
ping 1.1.1.1
```

```powershell
ping example.com
```

```powershell
route print
```

```powershell
Get-DnsClientServerAddress
```
