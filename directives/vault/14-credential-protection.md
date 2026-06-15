# Topic 14 — Credential Protection (WDigest / LSASS RunAsPPL)

**Category:** Credential Dumping Protection
**Risk for user:** NONE — Transparent change, no impact on daily use.
**Security risk (if not applied):** CRITICAL — Mimikatz extracts plaintext passwords from LSASS memory in seconds. MITRE ATT&CK Vector T1003.001.

**Affected Registry Keys:**
- `HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest` → `UseLogonCredential`
- `HKLM\SYSTEM\CurrentControlSet\Control\Lsa` → `RunAsPPL`

> **Context:** WDigest stores credentials in plaintext in LSASS for compatibility with HTTP Digest authentication. Disabling it forces LSASS to store only hashes. RunAsPPL adds protection to the LSASS process as Protected Process Light, preventing unsigned processes (like Mimikatz) from calling `OpenProcess` on it.

---

## Check

```powershell
$wdigest = (Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest" -ErrorAction SilentlyContinue).UseLogonCredential
$lsass = (Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -ErrorAction SilentlyContinue).RunAsPPL
return ($wdigest -eq 0) -and ($lsass -eq 1)
```

## Apply

```powershell
# Disable WDigest (prevents plaintext password storage)
$wdigestPath = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest"
if (!(Test-Path $wdigestPath)) { New-Item -Path $wdigestPath -Force | Out-Null }
Set-ItemProperty -Path $wdigestPath -Name "UseLogonCredential" -Value 0 -Type DWord -ErrorAction Stop

# Enable LSASS as Protected Process Light (RunAsPPL)
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "RunAsPPL" -Value 1 -Type DWord -ErrorAction Stop

Write-Host "    [!] RunAsPPL requires a reboot to take effect." -ForegroundColor Yellow
```

## Rollback

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest" -Name "UseLogonCredential" -Value 1 -Type DWord -ErrorAction SilentlyContinue
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "RunAsPPL" -Value 0 -Type DWord -ErrorAction SilentlyContinue
```

## Remediation Hints

- `RunAsPPL` is only effective after a reboot — the post-apply validation Check should inform that the status will be confirmed on the next boot.
- If `Set-ItemProperty` on `Lsa` returns Access Denied: use `reg add "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" /v RunAsPPL /t REG_DWORD /d 1 /f`.
- On systems with Secure Boot + UEFI, RunAsPPL can be configured via Credential Guard (more robust). For home environments, RunAsPPL via registry is already effective.
