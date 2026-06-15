# Topic 04 — Password Hardening

**Category:** Access Policy
**Risk for user:** HIGH — 15 min lockout after 5 failed attempts. Complex passwords mandatory.
**Security risk (if not applied):** HIGH — Weak passwords are the #1 compromise vector.

**Affected Registry Keys:** None directly (uses `secedit` for local security policy)

---

## Check

```powershell
$tempFile = "$env:TEMP\secpol_check_$(Get-Random).cfg"
secedit /export /cfg "$tempFile" /quiet
$content = Get-Content $tempFile -ErrorAction SilentlyContinue
Remove-Item $tempFile -Force -ErrorAction SilentlyContinue
$minLen = ($content | Select-String "MinimumPasswordLength\s*=\s*(\d+)").Matches.Groups[1].Value
$complexity = ($content | Select-String "PasswordComplexity\s*=\s*(\d+)").Matches.Groups[1].Value
return ([int]$minLen -ge 12) -and ([int]$complexity -eq 1)
```

## Apply

```powershell
$secpolContent = @"
[Unicode]
Unicode=yes
[System Access]
MinimumPasswordLength = 12
PasswordComplexity = 1
PasswordHistorySize = 24
MaximumPasswordAge = 90
MinimumPasswordAge = 1
LockoutBadCount = 5
ResetLockoutCount = 15
LockoutDuration = 15
[Version]
signature="`$CHICAGO`$"
Revision=1
"@
$secpolFile = [System.IO.Path]::Combine([System.IO.Path]::GetTempPath(), [System.IO.Path]::GetRandomFileName() + '.inf')
$secpolDb = [System.IO.Path]::Combine([System.IO.Path]::GetTempPath(), [System.IO.Path]::GetRandomFileName() + '.sdb')
$secpolContent | Out-File -Encoding Unicode -FilePath $secpolFile
secedit /configure /db "$secpolDb" /cfg "$secpolFile" /areas SECURITYPOLICY /quiet
Remove-Item $secpolFile -Force -ErrorAction SilentlyContinue
Remove-Item $secpolDb -Force -ErrorAction SilentlyContinue
gpupdate /force | Out-Null
```

## Rollback

```powershell
# Restore default Windows policy (minimum values)
net accounts /minpwlen:0 /maxpwage:unlimited /minpwage:0 /uniquepw:0 /lockoutthreshold:0
```

## Remediation Hints

- If `secedit /configure` fails with encoding error: re-save the `.inf` file with `Out-File -Encoding Default` (ANSI).
- If `gpupdate /force` hangs: run `gpupdate /force /boot` or simply ignore — the secedit policy is already applied.
- Direct alternative: `net accounts /minpwlen:12 /lockoutthreshold:5 /lockoutduration:15 /lockoutwindow:15`.
- **⚠️ Path with spaces:** `$env:TEMP` may contain spaces and break `secedit`. Always use `[System.IO.Path]::Combine(...)` to generate the safe path (already done in the Apply above).
