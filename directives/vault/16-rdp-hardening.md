# Topic 16 — RDP Hardening (Remote Desktop)

**Category:** Remote Access Security
**Risk for user:** MEDIUM — Forced NLA requires the RDP client to support Network Level Authentication (all modern clients do).
**Security risk (if not applied):** CRITICAL — RDP without NLA is exposed to pre-authentication brute force attacks and vulnerabilities like BlueKeep (CVE-2019-0708). MITRE ATT&CK Vector T1021.001.

**Affected Registry Keys:**
- `HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server` → `fDenyTSConnections`
- `HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp` → `UserAuthentication`

> **Dual Strategy:** If RDP is not used on the machine, completely disable it. If required, force NLA and restrict access only to the necessary user group.

---

## Check

```powershell
$rdpEnabled = (Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server" -ErrorAction SilentlyContinue).fDenyTSConnections
$nla = (Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -ErrorAction SilentlyContinue).UserAuthentication

# SECURE if: RDP disabled OR (RDP enabled AND NLA forced)
$rdpDisabled = $rdpEnabled -eq 1
$rdpSecured = ($rdpEnabled -eq 0) -and ($nla -eq 1)
return $rdpDisabled -or $rdpSecured
```

## Apply

```powershell
# Default strategy: completely disable RDP
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server" -Name "fDenyTSConnections" -Value 1 -Type DWord -ErrorAction Stop

# Also disable via Firewall
Disable-NetFirewallRule -DisplayGroup "Remote Desktop" -ErrorAction SilentlyContinue

# Force NLA even with RDP disabled (defense in depth)
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name "UserAuthentication" -Value 1 -Type DWord -ErrorAction SilentlyContinue

Write-Host "    RDP disabled. If you need remote access, use VPN + RDP restricted by IP." -ForegroundColor Cyan
```

## Rollback

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server" -Name "fDenyTSConnections" -Value 0 -Type DWord -ErrorAction SilentlyContinue
Enable-NetFirewallRule -DisplayGroup "Remote Desktop" -ErrorAction SilentlyContinue
```

## Remediation Hints

- If the user needs active RDP (e.g., legitimate remote administration): apply only NLA without disabling the service. Use `Set-ItemProperty ... UserAuthentication -Value 1` and keep `fDenyTSConnections = 0`.
- Additional security complement: change the default RDP port from `3389` to a non-standard port via `HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp` → `PortNumber`.
- Check if RDP is exposed externally: `Get-NetFirewallRule -DisplayGroup "Remote Desktop" | Select-Object Enabled, Profile`.
