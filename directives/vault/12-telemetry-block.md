# Topic 12 — Block OS Telemetry

**Category:** Privacy
**Risk for user:** LOW — May prevent participation in the Windows Insider Program.
**Security risk (if not applied):** LOW/MEDIUM — Diagnostic data is sent to Microsoft.

**Affected Registry Keys:**
- `HKLM\SOFTWARE\Policies\Microsoft\Windows\DataCollection` → `AllowTelemetry`

---

## Check

```powershell
$val = (Get-ItemProperty "HKLM:\SOFTWARE\Policies\Microsoft\Windows\DataCollection" -ErrorAction SilentlyContinue).AllowTelemetry
return $val -eq 0
```

## Apply

```powershell
$path = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\DataCollection"
if (!(Test-Path $path)) { New-Item -Path $path -Force | Out-Null }
Set-ItemProperty -Path $path -Name "AllowTelemetry" -Value 0 -ErrorAction Stop

# Also block via DiagTrack service (complementary to Topic 1)
Stop-Service -Name "DiagTrack" -Force -ErrorAction SilentlyContinue
Set-Service -Name "DiagTrack" -StartupType Disabled -ErrorAction SilentlyContinue
```

## Rollback

```powershell
Remove-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\DataCollection" -Name "AllowTelemetry" -ErrorAction SilentlyContinue
Set-Service -Name "DiagTrack" -StartupType Automatic -ErrorAction SilentlyContinue
```

## Remediation Hints

- In **Windows 10 Home** editions, the value `0` may be ignored by the system — the effective minimum is `1` (Basic). If the Check always returns false even after Apply, warn the user that the Home edition limits this control.
