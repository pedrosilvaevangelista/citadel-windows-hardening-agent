# Topic 13 — Block AutoRun / AutoPlay

**Category:** Physical Device Security
**Risk for user:** LOW — USBs and CDs will need to be opened manually.
**Security risk (if not applied):** HIGH — AutoRun is a classic infection vector via USB (e.g., infected pendrive).

**Affected Registry Keys:**
- `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer` → `NoDriveTypeAutoRun`
- `HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer` → `NoDriveTypeAutoRun`
- `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\AutoplayHandlers` → `DisableAutoplay`

---

## Check

```powershell
$val1 = (Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ErrorAction SilentlyContinue).NoDriveTypeAutoRun
$val2 = (Get-ItemProperty "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ErrorAction SilentlyContinue).NoDriveTypeAutoRun
$val3 = (Get-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\AutoplayHandlers" -ErrorAction SilentlyContinue).DisableAutoplay
return ($val1 -eq 255) -and ($val2 -eq 255) -and ($val3 -eq 1)
```

## Apply

```powershell
$policiesHKLM = "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer"
$policiesHKCU = "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer"
$autoplayHKCU = "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\AutoplayHandlers"

foreach ($p in @($policiesHKLM, $policiesHKCU, $autoplayHKCU)) {
    if (!(Test-Path $p)) { New-Item -Path $p -Force | Out-Null }
}

Set-ItemProperty -Path $policiesHKLM -Name "NoDriveTypeAutoRun" -Value 255 -Type DWord -ErrorAction Stop
Set-ItemProperty -Path $policiesHKCU -Name "NoDriveTypeAutoRun" -Value 255 -Type DWord -ErrorAction Stop
Set-ItemProperty -Path $autoplayHKCU -Name "DisableAutoplay" -Value 1 -Type DWord -ErrorAction Stop
```

## Rollback

```powershell
Remove-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer" -Name "NoDriveTypeAutoRun" -ErrorAction SilentlyContinue
Remove-ItemProperty -Path "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer" -Name "NoDriveTypeAutoRun" -ErrorAction SilentlyContinue
Remove-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\AutoplayHandlers" -Name "DisableAutoplay" -ErrorAction SilentlyContinue
```

## Remediation Hints

- If the HKLM key fails due to permissions: use `reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer" /v NoDriveTypeAutoRun /t REG_DWORD /d 255 /f`.
- The value `255` is equivalent to `0xFF` — blocks all drive types (fixed, removable, network, CD, etc.).
