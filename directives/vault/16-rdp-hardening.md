# Tópico 16 — Hardening de RDP (Remote Desktop)

**Categoria:** Segurança de Acesso Remoto
**Risco para o usuário:** MÉDIO — NLA forçado exige que o cliente RDP suporte autenticação de nível de rede (todos os clientes modernos suportam).
**Risco de segurança (não aplicar):** CRÍTICO — RDP sem NLA é exposto a ataques de força bruta pré-autenticação e vulnerabilidades do tipo BlueKeep (CVE-2019-0708). Vetor MITRE ATT&CK T1021.001.

**Chaves de Registro Afetadas:**
- `HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server` → `fDenyTSConnections`
- `HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp` → `UserAuthentication`

> **Estratégia dupla:** Se RDP não for utilizado na máquina, desabilitar completamente. Se for necessário, forçar NLA e restringir acesso apenas ao grupo de usuários necessário.

---

## Check

```powershell
$rdpEnabled = (Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server" -ErrorAction SilentlyContinue).fDenyTSConnections
$nla = (Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -ErrorAction SilentlyContinue).UserAuthentication

# SEGURO se: RDP desabilitado OU (RDP habilitado E NLA forçado)
$rdpDisabled = $rdpEnabled -eq 1
$rdpSecured = ($rdpEnabled -eq 0) -and ($nla -eq 1)
return $rdpDisabled -or $rdpSecured
```

## Apply

```powershell
# Estratégia padrão: desabilitar RDP completamente
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server" -Name "fDenyTSConnections" -Value 1 -Type DWord -ErrorAction Stop

# Desabilitar também via Firewall
Disable-NetFirewallRule -DisplayGroup "Remote Desktop" -ErrorAction SilentlyContinue

# Forçar NLA mesmo com RDP desabilitado (defense in depth)
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name "UserAuthentication" -Value 1 -Type DWord -ErrorAction SilentlyContinue

Write-Host "    RDP desabilitado. Se precisar de acesso remoto, use VPN + RDP restrito por IP." -ForegroundColor Cyan
```

## Rollback

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server" -Name "fDenyTSConnections" -Value 0 -Type DWord -ErrorAction SilentlyContinue
Enable-NetFirewallRule -DisplayGroup "Remote Desktop" -ErrorAction SilentlyContinue
```

## Remediation Hints

- Se o usuário precisar de RDP ativo (ex: administração remota legítima): aplicar apenas o NLA sem desabilitar o serviço. Usar `Set-ItemProperty ... UserAuthentication -Value 1` e manter `fDenyTSConnections = 0`.
- Complemento de segurança adicional: mudar a porta padrão do RDP de `3389` para uma porta não-padrão via `HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp` → `PortNumber`.
- Verificar se o RDP está exposto externamente: `Get-NetFirewallRule -DisplayGroup "Remote Desktop" | Select-Object Enabled, Profile`.
