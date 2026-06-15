# Topic 11 — PowerShell Advanced Logging

**Category:** Auditing and Intrusion Detection
**Risk for user:** LOW — Increases log size in the Event Viewer.
**Security risk (if not applied):** HIGH — Without logging, malicious PowerShell activity is invisible.

**Affected Registry Keys:**
- `HKLM\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging` → `EnableScriptBlockLogging`
- `HKLM\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging` → `EnableModuleLogging`
- `HKLM\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging\ModuleNames` → `*`

---

## Check

```powershell
$sb = (Get-ItemProperty "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" -ErrorAction SilentlyContinue).EnableScriptBlockLogging
$ml = (Get-ItemProperty "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging" -ErrorAction SilentlyContinue).EnableModuleLogging
return ($sb -eq 1) -and ($ml -eq 1)
```

## Apply

```powershell
$psPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell"
$sbPath = "$psPath\ScriptBlockLogging"
$mlPath = "$psPath\ModuleLogging"
$moPath = "$mlPath\ModuleNames"

foreach ($p in @($sbPath, $mlPath, $moPath)) {
    if (!(Test-Path $p)) { New-Item -Path $p -Force | Out-Null }
}

Set-ItemProperty -Path $sbPath -Name "EnableScriptBlockLogging" -Value 1 -ErrorAction Stop
Set-ItemProperty -Path $mlPath -Name "EnableModuleLogging" -Value 1 -ErrorAction Stop
New-ItemProperty -Path $moPath -Name "*" -Value "1" -PropertyType String -Force -ErrorAction Stop | Out-Null
```

## Rollback

```powershell
Remove-Item "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging" -Recurse -Force -ErrorAction SilentlyContinue
```

## Remediation Hints

- If `New-Item` creation fails in any subpath: use `reg add` for each key individually.
- The value name `"*"` in `ModuleNames` can cause conflict with PowerShell wildcards. Use `New-ItemProperty` (already done in Apply) instead of `Set-ItemProperty`.
