# Topic 03 — Windows Update Control

**Category:** Patch Management
**Risk for user:** MEDIUM — Requires discipline to install updates manually.
**Security risk (if not applied):** LOW — Auto-reboot may cause data loss in critical work.

**Affected Registry Keys:**
- `HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU` → `AUOptions`, `NoAutoRebootWithLoggedOnUsers`

---

## Check

```powershell
$val = (Get-ItemProperty "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" -ErrorAction SilentlyContinue).AUOptions
return $val -eq 2
```

## Apply

```powershell
$path = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU"
if (!(Test-Path $path)) { New-Item -Path $path -Force | Out-Null }
Set-ItemProperty -Path $path -Name "AUOptions" -Value 2 -ErrorAction Stop
Set-ItemProperty -Path $path -Name "NoAutoRebootWithLoggedOnUsers" -Value 1 -ErrorAction Stop
```

## Rollback

```powershell
$path = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU"
Remove-ItemProperty -Path $path -Name "AUOptions" -ErrorAction SilentlyContinue
Remove-ItemProperty -Path $path -Name "NoAutoRebootWithLoggedOnUsers" -ErrorAction SilentlyContinue
```

## Remediation Hints

- If the registry path cannot be created: check permissions on `HKLM:\SOFTWARE\Policies\Microsoft\Windows`. Try `takeown` on the parent key.
- In domain machines, server GPO can overwrite this value. Warn the user that the configuration may not persist.
