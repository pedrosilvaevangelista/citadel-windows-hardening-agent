# Topic 01 — Critical Services

**Category:** Attack Surface Reduction
**Risk for user:** Xbox App and Fax stop working. Telemetry disabled.
**Security risk (if not applied):** MEDIUM — RemoteRegistry allows remote registry access.

**Affected Registry Keys:** None (services only)

---

## Check

```powershell
$svcs = @("RemoteRegistry", "Fax", "DiagTrack", "XblGameSave")
$allDisabled = $true
foreach ($s in $svcs) {
    $svc = Get-Service -Name $s -ErrorAction SilentlyContinue
    if ($null -ne $svc -and $svc.StartType -ne 'Disabled') { $allDisabled = $false }
}
return $allDisabled
```

## Apply

```powershell
$services = @("RemoteRegistry", "Fax", "XblGameSave", "DiagTrack", "XboxNetApiSvc", "MapsBroker", "WerSvc")
foreach ($s in $services) {
    $svc = Get-Service -Name $s -ErrorAction SilentlyContinue
    if ($null -ne $svc) {
        Stop-Service -Name $s -Force -ErrorAction SilentlyContinue
        Set-Service -Name $s -StartupType Disabled -ErrorAction Stop
    }
}
```

## Rollback

```powershell
$services = @("RemoteRegistry", "Fax", "XblGameSave", "DiagTrack", "XboxNetApiSvc", "MapsBroker", "WerSvc")
foreach ($s in $services) {
    $svc = Get-Service -Name $s -ErrorAction SilentlyContinue
    if ($null -ne $svc) { Set-Service -Name $s -StartupType Manual -ErrorAction SilentlyContinue }
}
```

## Remediation Hints

- If `Set-Service` returns "Access Denied", try `sc.exe config [name] start= disabled` as an alternative.
- If the service does not exist (e.g., newer Windows version removed XboxNetApiSvc), silently ignore and continue.
