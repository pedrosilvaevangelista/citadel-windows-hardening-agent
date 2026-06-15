# Topic 05 — Firewall (All Profiles)

**Category:** Network Traffic Control
**Risk for user:** LOW — May block local apps without a configured rule.
**Security risk (if not applied):** CRITICAL — Disabled firewall exposes ports to the entire network.

**Affected Registry Keys:** None (managed by firewall cmdlets)

---

## Check

```powershell
$enabled = (Get-NetFirewallProfile -Profile Domain,Private,Public | Where-Object {$_.Enabled -eq 'True'}).Count
return $enabled -eq 3
```

## Apply

```powershell
Set-NetFirewallProfile -Profile Domain,Private,Public -Enabled True -ErrorAction Stop
```

## Rollback

```powershell
Set-NetFirewallProfile -Profile Domain,Private,Public -Enabled False -ErrorAction SilentlyContinue
```

## Remediation Hints

- If `Set-NetFirewallProfile` fails: use `netsh advfirewall set allprofiles state on` as an alternative.
