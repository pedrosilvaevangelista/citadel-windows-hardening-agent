# Topic 08 — Secure DNS (1.1.1.1 Cloudflare)

**Category:** Privacy and Network
**Risk for user:** MEDIUM — If Cloudflare goes down, the internet stops. In corporate networks, breaks internal domain resolution.
**Security risk (if not applied):** MEDIUM — Default ISP DNS can be intercepted (DNS Spoofing).

**Affected Registry Keys:** None (managed by network cmdlets)

> **⚠️ AUTOMATIC SKIP IN DOMAIN**: If the Pre-Flight detects that the machine is in an Active Directory (`PartOfDomain = True`), this topic must be marked as `SKIP_DOMAIN` and should not be applied without explicit user confirmation.

---

## Check

```powershell
$addresses = (Get-DnsClientServerAddress -AddressFamily IPv4 -ErrorAction SilentlyContinue).ServerAddresses
return $addresses -contains "1.1.1.1"
```

## Apply

```powershell
Get-NetAdapter | Where-Object Status -eq 'Up' | Set-DnsClientServerAddress -ServerAddresses ("1.1.1.1", "1.0.0.1") -ErrorAction Stop
```

## Rollback

```powershell
Get-NetAdapter | Where-Object Status -eq 'Up' | Set-DnsClientServerAddress -ResetServerAddresses -ErrorAction SilentlyContinue
```

## Remediation Hints

- If `Set-DnsClientServerAddress` fails: use `netsh interface ip set dns "[adapter_name]" static 1.1.1.1` for each active adapter.
- To get the adapter names: `Get-NetAdapter | Where-Object Status -eq 'Up' | Select-Object Name`.
