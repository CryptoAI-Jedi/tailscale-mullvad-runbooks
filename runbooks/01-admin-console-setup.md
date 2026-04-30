# 01 - Admin Console Setup

## Purpose

This runbook explains how to enable Mullvad VPN exit nodes inside Tailscale.

There are two main ways to manage Mullvad access:

1. Tailscale admin console
2. Tailnet policy file

Use one method. Do not manage Mullvad access from both places at the same time.

## Requirements

- Tailscale account
- Admin access to the tailnet
- Mullvad VPN add-on purchased through Tailscale
- Target device joined to the correct tailnet
- Tailscale client version `1.48.2` or newer

## Method 1: Enable Mullvad in Tailscale Admin Console

1. Open the Tailscale admin console.
2. Go to **Settings**.
3. Find **Mullvad VPN**.
4. Select **Configure**.
5. Purchase or enable the Mullvad VPN add-on.
6. Select the devices that should be allowed to use Mullvad exit nodes.
7. Save changes.

## Verify Device Access

On the target device, run:

```bash
tailscale status
```

Then list exit nodes:

```bash
tailscale exit-node list
```

Expected result:

```text
Mullvad exit nodes appear in the exit-node list.
```

If no Mullvad exit nodes appear, confirm:

- The Mullvad add-on is active.
- The device was granted Mullvad access.
- The device is logged into the correct Tailscale account.
- The device is joined to the correct tailnet.
- The Tailscale client is updated.
- The device has reconnected after access was granted.

## Method 2: Enable Mullvad with Tailnet Policy File

Use this method if you manage your tailnet through a policy file.

Example: grant Mullvad access to one user.

```json
"nodeAttrs": [
  {
    "target": ["user@example.com"],
    "attr": [
      "mullvad"
    ]
  }
]
```

Example: grant Mullvad access to a group.

```json
"groups": {
  "group:mullvad": [
    "user1@example.com",
    "user2@example.com"
  ]
},
"nodeAttrs": [
  {
    "target": ["group:mullvad"],
    "attr": [
      "mullvad"
    ]
  }
]
```

## Important Policy Notes

Use either:

```text
Admin console management
```

Or:

```text
Policy file management
```

Do not use both simultaneously.

If using the policy file, manage all Mullvad access through the policy file.

## Verify Policy File Access

After saving the policy file, reconnect the device.

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

Then list Mullvad exit nodes:

```bash
tailscale exit-node list
```

Expected result:

```text
Mullvad exit nodes are visible.
```

## Check Tailscale Version

Linux/macOS/Windows:

```bash
tailscale version
```

Recommended:

```text
Tailscale v1.48.3 or later
```

## Check Device Status

```bash
tailscale status
```

Look for:

- Device is online
- Correct tailnet
- Expected user/account
- No authentication errors
- No expired node key warnings

## License Slot Notes

Mullvad access through Tailscale is licensed by device slots.

If all slots are already used:

- New devices may not get Mullvad access.
- Existing devices may still work.
- You may need to remove unused devices from Mullvad access.
- You may need to purchase additional device slots.

## Remove Device Access

If managing through the admin console:

1. Open the Tailscale admin console.
2. Go to Mullvad VPN settings.
3. Remove the device from Mullvad access.
4. Save changes.

If managing through policy file:

1. Remove the user, group, or node from the `nodeAttrs` Mullvad target.
2. Save the policy file.
3. Reconnect the device.

## Reconnect Device

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

## Confirm Mullvad Exit Nodes

```bash
tailscale exit-node list
```

Expected result:

```text
Mullvad exit nodes are visible.
```

## Basic Functional Test

Select a Mullvad exit node.

Linux/macOS:

```bash
sudo tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net
```

Windows PowerShell:

```powershell
tailscale set --exit-node=us-nyc-wg-001.mullvad.ts.net
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

Check IP geolocation.

Linux/macOS:

```bash
curl ipinfo.io
```

Windows PowerShell:

```powershell
curl.exe ipinfo.io
```

Disable exit node.

Linux/macOS:

```bash
sudo tailscale set --exit-node=
```

Windows PowerShell:

```powershell
tailscale set --exit-node=
```

## Troubleshooting: Mullvad Nodes Do Not Appear

Check Tailscale status:

```bash
tailscale status
```

Check Tailscale version:

```bash
tailscale version
```

List exit nodes:

```bash
tailscale exit-node list
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

Then list exit nodes again:

```bash
tailscale exit-node list
```

## Troubleshooting Checklist

If Mullvad exit nodes still do not appear:

- Confirm the Mullvad add-on is paid and active.
- Confirm the target device is selected for Mullvad access.
- Confirm the device is not logged into another tailnet.
- Confirm the device has reconnected.
- Confirm Tailscale is updated.
- Confirm you are not mixing admin console and policy file management.
- Confirm Tailnet Lock is not blocking unsigned Mullvad nodes.
