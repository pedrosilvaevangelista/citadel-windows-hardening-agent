# Topic 06 — Block WSH (.VBS / .JS)

**Category:** Script Execution Control
**Risk for user:** HIGH — Legacy installers and automation scripts in VBScript or JScript will stop working.
**Security risk (if not applied):** HIGH — WSH is a classic malware vector via email (e.g., disguised .vbs).

**Affected Registry Keys:**
- `HKLM\SOFTWARE\Microsoft\Windows Script Host\Settings` → `Enabled`

---

## Check

```powershell
$val = (Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows Script Host\Settings" -ErrorAction SilentlyContinue).Enabled
return $val -eq 0
```

## Apply

```powershell
$path = "HKLM:\SOFTWARE\Microsoft\Windows Script Host\Settings"
if (!(Test-Path $path)) { New-Item -Path $path -Force | Out-Null }
Set-ItemProperty -Path $path -Name "Enabled" -Value 0 -ErrorAction Stop
```

## Rollback

```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows Script Host\Settings" -Name "Enabled" -Value 1 -ErrorAction SilentlyContinue
```

## Remediation Hints

- If the key does not exist and `New-Item` fails: use `reg add "HKLM\SOFTWARE\Microsoft\Windows Script Host\Settings" /v Enabled /t REG_DWORD /d 0 /f`.
