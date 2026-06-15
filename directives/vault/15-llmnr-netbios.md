# Topic 15 — Disable LLMNR and NetBIOS

**Category:** Local Network MITM Protection
**Risk for user:** LOW — May break name resolution on very old networks without configured DNS.
**Security risk (if not applied):** CRITICAL — Responder tool exploits LLMNR/NetBIOS to capture NTLMv2 hashes from any machine on the local network. MITRE ATT&CK Vector T1557.001.

**Affected Registry Keys:**
- `HKLM\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient` → `EnableMulticast`
- `HKLM\SYSTEM\CurrentControlSet\Services\NetBT\Parameters\Interfaces\Tcpip_{GUID}` → `NetbiosOptions` (fallback)

> **Context:** LLMNR (Link-Local Multicast Name Resolution) and NetBIOS are name resolution fallback protocols. When a hostname is not found in DNS, Windows shouts on the network via LLMNR/NetBIOS — any attacker on the same network can answer pretending to be the legitimate server and capture the user's authentication hash.

---

## Check

```powershell
# Check LLMNR via policy registry
$llmnr = (Get-ItemProperty "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient" -ErrorAction SilentlyContinue).EnableMulticast

# Check NetBIOS via WMI
$netbiosDisabled = $true
$adapters = Get-WmiObject -Class Win32_NetworkAdapterConfiguration -Filter "IPEnabled=True" -ErrorAction SilentlyContinue
foreach ($adapter in $adapters) {
    if ($adapter.TcpipNetbiosOptions -ne 2) { $netbiosDisabled = $false }
}

return ($llmnr -eq 0) -and $netbiosDisabled
```

## Apply

```powershell
# Disable LLMNR via local GPO
$dnsPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient"
if (!(Test-Path $dnsPath)) { New-Item -Path $dnsPath -Force | Out-Null }
Set-ItemProperty -Path $dnsPath -Name "EnableMulticast" -Value 0 -Type DWord -ErrorAction Stop

# Disable NetBIOS on all active adapters (TcpipNetbiosOptions=2 = Disable)
$adapters = Get-WmiObject -Class Win32_NetworkAdapterConfiguration -Filter "IPEnabled=True" -ErrorAction SilentlyContinue
foreach ($adapter in $adapters) {
    $result = $adapter.SetTcpipNetbios(2)
    if ($result.ReturnValue -ne 0) {
        Write-Host "    [!] Warning: NetBIOS may not have been disabled on adapter: $($adapter.Description)" -ForegroundColor Yellow
    }
}
```

## Rollback

```powershell
# Re-enable LLMNR
Remove-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient" -Name "EnableMulticast" -ErrorAction SilentlyContinue

# Re-enable NetBIOS (0 = Default, lets DHCP decide)
$adapters = Get-WmiObject -Class Win32_NetworkAdapterConfiguration -Filter "IPEnabled=True" -ErrorAction SilentlyContinue
foreach ($adapter in $adapters) { $adapter.SetTcpipNetbios(0) | Out-Null }
```

## Remediation Hints

- If `Get-WmiObject` fails (corrupted WMI service): restart WMI with `net stop winmgmt && net start winmgmt` and try again.
- If `SetTcpipNetbios(2)` returns an error code other than `0`: try via registry in `HKLM:\SYSTEM\CurrentControlSet\Services\NetBT\Parameters\Interfaces\Tcpip_{GUID}` with `NetbiosOptions = 2` for each GUID.
- To get adapter GUIDs: `Get-ChildItem "HKLM:\SYSTEM\CurrentControlSet\Services\NetBT\Parameters\Interfaces"`.
