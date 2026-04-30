# Incident Playbooks

Operational incident playbooks for troubleshooting Tailscale + Mullvad exit node issues across Linux, macOS, and Windows.

These playbooks are designed for fast triage during connectivity failures, DNS failures, routing issues, and exit node misconfiguration.

## Scope

This document covers incident response for:

- No internet after enabling a Mullvad exit node
- DNS resolution failures
- Tailnet reachable but internet unavailable
- Exit node enabled but traffic not using Mullvad
- Device cannot reach other tailnet peers
- Slow or unstable connectivity through an exit node
- Reset and recovery procedures

## Assumptions

- Tailscale is installed and authenticated.
- Mullvad exit nodes are enabled in the Tailscale admin console.
- The device has previously worked with Tailscale or has completed initial setup.
- The operator has local admin/sudo access.
- Commands are executed from a local terminal or PowerShell session.

## Severity Levels

| Severity | Description | Example |
| --- | --- | --- |
| SEV1 | Total loss of internet or production-critical access | No internet after enabling exit node |
| SEV2 | Partial connectivity loss | Tailnet works, public internet fails |
| SEV3 | Degraded performance | Exit node works but latency is high |
| SEV4 | Non-urgent configuration issue | Wrong exit node selected |

## First Response Checklist

Before deep troubleshooting, collect the current state.

### Linux/macOS

    tailscale status
    tailscale exit-node list
    tailscale ip -4
    tailscale netcheck

Check DNS:

    nslookup example.com
    dig example.com

Check public IP:

    curl ifconfig.me

Check routing on Linux:

    ip route

Check routing on macOS:

    netstat -rn

### Windows PowerShell

    tailscale status
    tailscale exit-node list
    tailscale ip -4
    tailscale netcheck

Check DNS:

    nslookup example.com

Check public IP:

    curl ifconfig.me

Check routing:

    route print

## Playbook 1: No Internet After Enabling Exit Node

### Symptoms

- Browser pages do not load.
- `ping 1.1.1.1` fails.
- DNS lookups fail.
- Tailnet may or may not still work.
- Issue starts immediately after selecting a Mullvad exit node.

### Likely Causes

- Exit node is unhealthy or unreachable.
- Local DNS is broken.
- Tailscale routing state is stale.
- Local firewall blocks Tailscale traffic.
- Device is enforcing exit node routing but cannot reach the selected exit node.

### Triage Steps

#### 1. Confirm Tailscale State

    tailscale status

Look for:

- Device is logged in.
- Exit node is selected.
- No obvious offline/error state.

#### 2. Disable Exit Node Immediately

Linux/macOS:

    sudo tailscale set --exit-node=

Windows PowerShell:

    tailscale set --exit-node=

Then test internet access on Linux/macOS:

    ping 1.1.1.1
    curl https://example.com

Then test internet access on Windows:

    ping 1.1.1.1
    curl https://example.com

#### 3. If Internet Returns

The issue is likely with:

- Selected Mullvad exit node
- Exit node routing
- DNS while using exit node

Select a different Mullvad exit node:

    tailscale exit-node list

Then enable another node on Linux/macOS:

    sudo tailscale set --exit-node=<exit-node-name>

Enable another node on Windows:

    tailscale set --exit-node=<exit-node-name>

#### 4. If Internet Does Not Return

Restart Tailscale.

Linux:

    sudo systemctl restart tailscaled

macOS:

    sudo tailscale down
    sudo tailscale up

Windows PowerShell:

    Restart-Service Tailscale

Retest on Linux/macOS:

    ping 1.1.1.1
    curl https://example.com

Retest on Windows:

    ping 1.1.1.1
    curl https://example.com

### Resolution

Use a different exit node or reset Tailscale routing state.

### Escalation Criteria

Escalate if:

- Multiple exit nodes fail.
- Internet remains down after disabling the exit node.
- Tailscale cannot restart cleanly.
- `tailscale netcheck` shows blocked UDP or DERP-only fallback.

## Playbook 2: DNS Broken but IP Connectivity Works

### Symptoms

- `ping 1.1.1.1` works.
- `ping example.com` fails.
- Browser cannot load domain names.
- `curl https://1.1.1.1` may respond, but normal websites fail.
- Tailnet IPs may still work.

### Likely Causes

- DNS resolver conflict.
- MagicDNS issue.
- Local DNS cache corruption.
- Exit node DNS handling conflict.
- Local VPN/DNS profile conflict.

### Triage Steps

#### 1. Test Raw IP Connectivity

    ping 1.1.1.1

If this works, routing is probably functional.

#### 2. Test DNS Resolution

    nslookup example.com

Linux/macOS:

    dig example.com

Windows:

    nslookup example.com

#### 3. Check Tailscale DNS Status

    tailscale status

Check whether MagicDNS is enabled in the admin console if applicable.

#### 4. Flush DNS Cache

Linux with systemd-resolved:

    sudo resolvectl flush-caches

macOS:

    sudo dscacheutil -flushcache
    sudo killall -HUP mDNSResponder

Windows PowerShell:

    ipconfig /flushdns

#### 5. Disable Exit Node and Retest DNS

Linux/macOS:

    sudo tailscale set --exit-node=

Windows:

    tailscale set --exit-node=

Retest:

    nslookup example.com

#### 6. Re-enable Exit Node

    tailscale exit-node list

Linux/macOS:

    sudo tailscale set --exit-node=<exit-node-name>

Windows:

    tailscale set --exit-node=<exit-node-name>

### Resolution

DNS failures are usually resolved by:

- Flushing local DNS cache
- Disabling and re-enabling the exit node
- Selecting another Mullvad exit node
- Removing conflicting local VPN/DNS tools

### Escalation Criteria

Escalate if:

- DNS fails across multiple exit nodes.
- DNS fails even with exit node disabled.
- Device uses corporate DNS, MDM, or another VPN profile.
- `/etc/resolv.conf` or OS DNS settings are being overwritten repeatedly.

## Playbook 3: Tailnet Reachable but Internet Dead

### Symptoms

- You can reach other tailnet devices.
- You can SSH/RDP into tailnet peers.
- Public internet does not work.
- Issue occurs only when an exit node is selected.

### Likely Causes

- Exit node selected but not routing public internet correctly.
- Exit node path is unhealthy.
- Local traffic is being routed into Tailscale, but exit path is failing.
- DNS is functional but default route is broken.

### Triage Steps

#### 1. Confirm Tailnet Peer Reachability

    tailscale status

Ping another tailnet device:

    ping <tailnet-peer-ip>

#### 2. Test Public IP Connectivity

    ping 1.1.1.1

If tailnet works but public IP fails, isolate exit node routing.

#### 3. Disable Exit Node

Linux/macOS:

    sudo tailscale set --exit-node=

Windows:

    tailscale set --exit-node=

Retest internet:

    ping 1.1.1.1
    curl https://example.com

#### 4. Select a Different Exit Node

    tailscale exit-node list

Linux/macOS:

    sudo tailscale set --exit-node=<different-exit-node-name>

Windows:

    tailscale set --exit-node=<different-exit-node-name>

#### 5. Check Network Path

    tailscale netcheck

Look for:

- UDP blocked
- DERP fallback only
- High latency
- Port mapping unavailable

### Resolution

Use a healthy exit node and confirm the local network allows Tailscale connectivity.

### Escalation Criteria

Escalate if:

- Tailnet works but all exit nodes fail.
- DERP-only fallback causes unusable performance.
- Local firewall blocks WireGuard UDP.
- Network blocks VPN-like traffic.

## Playbook 4: Exit Node Enabled but Public IP Does Not Change

### Symptoms

- Tailscale shows an exit node is selected.
- Internet works.
- `curl ifconfig.me` still shows the local ISP/public IP.
- Websites do not appear to route through Mullvad.

### Likely Causes

- Exit node setting did not apply.
- Split tunneling or local route preference is overriding traffic.
- Tailscale client state is stale.
- Exit node was selected in GUI but not active.
- Browser or app is using a separate proxy/VPN path.

### Triage Steps

#### 1. Confirm Current Public IP

Linux/macOS:

    curl ifconfig.me

Windows:

    curl ifconfig.me

#### 2. Confirm Exit Node Selection

    tailscale status

Check whether the selected exit node is shown.

#### 3. Reapply Exit Node

Linux/macOS:

    sudo tailscale set --exit-node=
    sudo tailscale set --exit-node=<exit-node-name>

Windows:

    tailscale set --exit-node=
    tailscale set --exit-node=<exit-node-name>

#### 4. Restart Tailscale

Linux:

    sudo systemctl restart tailscaled

macOS:

    sudo tailscale down
    sudo tailscale up

Windows PowerShell:

    Restart-Service Tailscale

#### 5. Retest Public IP

    curl ifconfig.me

### Resolution

Reapply the exit node or restart Tailscale.

### Escalation Criteria

Escalate if:

- Exit node shows active but traffic never routes through it.
- Other VPN/proxy software is installed.
- Corporate endpoint security controls routing.
- OS network profiles prevent route updates.

## Playbook 5: Device Cannot Reach Other Tailnet Peers

### Symptoms

- Internet works.
- Exit node may or may not work.
- Cannot reach other devices in the tailnet.
- SSH/RDP/HTTP to tailnet IP fails.
- `tailscale status` shows peers but connections fail.

### Likely Causes

- Peer is offline.
- ACL policy blocks access.
- Local firewall blocks inbound traffic.
- Service is not listening on target device.
- Device is authenticated into the wrong tailnet.

### Triage Steps

#### 1. Check Tailnet Status

    tailscale status

Confirm:

- Current device is logged in.
- Target peer is listed.
- Target peer is not offline.

#### 2. Confirm Local Tailscale IP

    tailscale ip -4

#### 3. Ping Target Peer

    ping <tailnet-peer-ip>

#### 4. Test Target Service

SSH example:

    ssh user@<tailnet-peer-ip>

HTTP example:

    curl http://<tailnet-peer-ip>:<port>

#### 5. Check ACLs

Review Tailscale admin console ACL policy.

Confirm that the source user/device is allowed to reach the destination device and port.

#### 6. Check Target Firewall

Linux target:

    sudo ufw status
    sudo iptables -L

Windows target:

    Get-NetFirewallProfile

### Resolution

Fix peer availability, ACL policy, local firewall, or service binding.

### Escalation Criteria

Escalate if:

- Peer appears online but all ports fail.
- ACL policy appears correct but traffic is blocked.
- The device joined the wrong tailnet.
- Admin console shows stale or duplicate devices.

## Playbook 6: Slow or Unstable Exit Node Connectivity

### Symptoms

- Internet works through exit node.
- Speed is poor.
- Latency is high.
- Video calls, SSH, or downloads are unstable.
- Issue changes depending on selected exit node.

### Likely Causes

- Distant exit node region.
- DERP relay fallback.
- Local ISP throttling or packet loss.
- Wi-Fi instability.
- Exit node congestion.

### Triage Steps

#### 1. Run Netcheck

    tailscale netcheck

Look for:

- Direct connection available
- DERP region
- UDP status
- Latency

#### 2. Check Basic Latency

    ping 1.1.1.1

#### 3. Check Public IP and Region

    curl ifconfig.me

Use a browser-based IP lookup if region confirmation is needed.

#### 4. Change Exit Node Region

    tailscale exit-node list

Select a geographically closer or less congested node.

Linux/macOS:

    sudo tailscale set --exit-node=<exit-node-name>

Windows:

    tailscale set --exit-node=<exit-node-name>

#### 5. Test Without Exit Node

Linux/macOS:

    sudo tailscale set --exit-node=

Windows:

    tailscale set --exit-node=

Compare performance with and without the exit node.

### Resolution

Use a closer, healthier exit node and avoid DERP-only paths when possible.

### Escalation Criteria

Escalate if:

- All exit nodes are slow.
- `tailscale netcheck` shows UDP blocked.
- Local network packet loss exists without Tailscale.
- Performance fails only on one ISP or Wi-Fi network.

## Playbook 7: Full Reset and Recovery

Use this when normal troubleshooting fails.

### Linux

Disable exit node:

    sudo tailscale set --exit-node=

Restart Tailscale:

    sudo systemctl restart tailscaled

Bring Tailscale down and up:

    sudo tailscale down
    sudo tailscale up

Check status:

    tailscale status
    tailscale netcheck

### macOS

Disable exit node:

    sudo tailscale set --exit-node=

Restart Tailscale session:

    sudo tailscale down
    sudo tailscale up

Flush DNS:

    sudo dscacheutil -flushcache
    sudo killall -HUP mDNSResponder

Check status:

    tailscale status
    tailscale netcheck

### Windows PowerShell

Disable exit node:

    tailscale set --exit-node=

Restart service:

    Restart-Service Tailscale

Flush DNS:

    ipconfig /flushdns

Check status:

    tailscale status
    tailscale netcheck

## Incident Notes Template

Use this template when documenting an incident.

### Incident Report Template

    # Incident Report: <Short Title>

    ## Summary

    - Date:
    - Device:
    - OS:
    - Network:
    - Exit node:
    - Severity:
    - Status:

    ## Symptoms

    -

    ## Timeline

    | Time | Event |
    | --- | --- |
    |  |  |

    ## Commands Run

    -

    ## Findings

    -

    ## Root Cause

    -

    ## Resolution

    -

    ## Follow-Up Actions

    -

## Quick Decision Tree

    Internet broken after enabling exit node?
    ├── Can you ping 1.1.1.1?
    │   ├── No
    │   │   ├── Disable exit node
    │   │   ├── Restart Tailscale
    │   │   └── Try different exit node
    │   └── Yes
    │       ├── Can you resolve example.com?
    │       │   ├── No
    │       │   │   ├── Flush DNS cache
    │       │   │   ├── Disable/re-enable exit node
    │       │   │   └── Check MagicDNS/local DNS conflict
    │       │   └── Yes
    │       │       ├── Check public IP
    │       │       ├── Confirm exit node is active
    │       │       └── Reapply exit node
    │
    ├── Tailnet works but internet fails?
    │   ├── Exit node routing issue
    │   ├── Try another exit node
    │   └── Run tailscale netcheck
    │
    └── Everything slow?
        ├── Run tailscale netcheck
        ├── Check DERP fallback
        ├── Try closer exit node
        └── Compare with exit node disabled

## Operator Notes

- Always disable the exit node first if the user has total internet loss.
- Separate DNS failures from routing failures.
- Always test raw IP connectivity before testing domain names.
- Always compare behavior with and without the exit node.
- Document the selected exit node and public IP before and after changes.
- If multiple exit nodes fail, suspect local network, DNS, firewall, or Tailscale admin policy.
