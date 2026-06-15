# Tópico 15 — Desabilitar LLMNR e NetBIOS

**Categoria:** Proteção Contra MITM em Rede Local
**Risco para o usuário:** BAIXO — Pode quebrar resolução de nomes em redes muito antigas sem DNS configurado.
**Risco de segurança (não aplicar):** CRÍTICO — A ferramenta Responder explora LLMNR/NetBIOS para capturar hashes NTLMv2 de qualquer máquina na rede local. Vetor MITRE ATT&CK T1557.001.

**Chaves de Registro Afetadas:**
- `HKLM\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient` → `EnableMulticast`
- `HKLM\SYSTEM\CurrentControlSet\Services\NetBT\Parameters\Interfaces\Tcpip_{GUID}` → `NetbiosOptions` (fallback)

> **Contexto:** LLMNR (Link-Local Multicast Name Resolution) e NetBIOS são protocolos de fallback de resolução de nomes. Quando um hostname não é encontrado no DNS, o Windows grita na rede via LLMNR/NetBIOS — qualquer atacante na mesma rede pode responder se passando pelo servidor legítimo e capturar o hash de autenticação do usuário.

---

## Check

```powershell
# Verificar LLMNR via registro de política
$llmnr = (Get-ItemProperty "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient" -ErrorAction SilentlyContinue).EnableMulticast

# Verificar NetBIOS via WMI
$netbiosDisabled = $true
$adapters = Get-WmiObject -Class Win32_NetworkAdapterConfiguration -Filter "IPEnabled=True" -ErrorAction SilentlyContinue
foreach ($adapter in $adapters) {
    if ($adapter.TcpipNetbiosOptions -ne 2) { $netbiosDisabled = $false }
}

return ($llmnr -eq 0) -and $netbiosDisabled
```

## Apply

```powershell
# Desabilitar LLMNR via GPO local
$dnsPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient"
if (!(Test-Path $dnsPath)) { New-Item -Path $dnsPath -Force | Out-Null }
Set-ItemProperty -Path $dnsPath -Name "EnableMulticast" -Value 0 -Type DWord -ErrorAction Stop

# Desabilitar NetBIOS em todos os adaptadores ativos (TcpipNetbiosOptions=2 = Disable)
$adapters = Get-WmiObject -Class Win32_NetworkAdapterConfiguration -Filter "IPEnabled=True" -ErrorAction SilentlyContinue
foreach ($adapter in $adapters) {
    $result = $adapter.SetTcpipNetbios(2)
    if ($result.ReturnValue -ne 0) {
        Write-Host "    [!] Aviso: NetBIOS pode nao ter sido desabilitado no adaptador: $($adapter.Description)" -ForegroundColor Yellow
    }
}
```

## Rollback

```powershell
# Reabilitar LLMNR
Remove-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient" -Name "EnableMulticast" -ErrorAction SilentlyContinue

# Reabilitar NetBIOS (0 = Default, deixa o DHCP decidir)
$adapters = Get-WmiObject -Class Win32_NetworkAdapterConfiguration -Filter "IPEnabled=True" -ErrorAction SilentlyContinue
foreach ($adapter in $adapters) { $adapter.SetTcpipNetbios(0) | Out-Null }
```

## Remediation Hints

- Se `Get-WmiObject` falhar (serviço WMI corrompido): reiniciar WMI com `net stop winmgmt && net start winmgmt` e tentar novamente.
- Se `SetTcpipNetbios(2)` retornar código de erro diferente de `0`: tentar via registro em `HKLM:\SYSTEM\CurrentControlSet\Services\NetBT\Parameters\Interfaces\Tcpip_{GUID}` com `NetbiosOptions = 2` para cada GUID.
- Obter GUIDs dos adaptadores: `Get-ChildItem "HKLM:\SYSTEM\CurrentControlSet\Services\NetBT\Parameters\Interfaces"`.
