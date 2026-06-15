# Topic 02 — SMBv1 Protocol

**Category:** Legacy Protocols
**Risk for user:** LOW — NAS/printers older than 15 years may lose connection.
**Security risk (if not applied):** CRITICAL — WannaCry/EternalBlue attack vector.

**Affected Registry Keys:**
- `HKLM\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters` → `SMB1`

---

## Check

```powershell
$feature = (Get-WindowsOptionalFeature -Online -FeatureName "SMB1Protocol" -ErrorAction SilentlyContinue).State
$reg = (Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" -Name "SMB1" -ErrorAction SilentlyContinue).SMB1
return ($feature -eq 'Disabled') -and ($reg -eq 0)
```

## Apply

```powershell
Set-SmbServerConfiguration -EnableSMB1Protocol $false -Force -ErrorAction Stop
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" -Name "SMB1" -Value 0 -Force -ErrorAction Stop
Disable-WindowsOptionalFeature -Online -FeatureName "SMB1Protocol" -NoRestart -WarningAction SilentlyContinue -ErrorAction Stop | Out-Null
```

## Rollback

```powershell
Set-SmbServerConfiguration -EnableSMB1Protocol $true -Force -ErrorAction SilentlyContinue
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" -Name "SMB1" -Value 1 -Force -ErrorAction SilentlyContinue
Enable-WindowsOptionalFeature -Online -FeatureName "SMB1Protocol" -NoRestart -ErrorAction SilentlyContinue | Out-Null
```

## Remediation Hints

- If `Get-WindowsOptionalFeature` fails (CBS service unavailable): use `dism /online /Disable-Feature /FeatureName:SMB1Protocol /NoRestart` as an alternative.
- If `Set-SmbServerConfiguration` returns permission error: forcing via registry is enough — the registry check already validates this.
- **⚠️ Post-apply warning:** `Disable-WindowsOptionalFeature` may return `RestartNeeded = True`. In this case, the Check will return false even with the correct configuration. Warn the user that the status will only be confirmed after reboot.
