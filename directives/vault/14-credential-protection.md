# Tópico 14 — Proteção de Credenciais (WDigest / LSASS RunAsPPL)

**Categoria:** Proteção Contra Credential Dumping
**Risco para o usuário:** NENHUM — Mudança transparente, sem impacto em uso diário.
**Risco de segurança (não aplicar):** CRÍTICO — Mimikatz extrai senhas em texto plano da memória LSASS em segundos. Vetor MITRE ATT&CK T1003.001.

**Chaves de Registro Afetadas:**
- `HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest` → `UseLogonCredential`
- `HKLM\SYSTEM\CurrentControlSet\Control\Lsa` → `RunAsPPL`

> **Contexto:** WDigest armazena credenciais em texto claro no LSASS para compatibilidade com autenticação HTTP Digest. Desabilitá-lo força o LSASS a armazenar apenas hashes. RunAsPPL adiciona proteção ao processo LSASS como Protected Process Light, impedindo que processos não assinados (como Mimikatz) façam `OpenProcess` nele.

---

## Check

```powershell
$wdigest = (Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest" -ErrorAction SilentlyContinue).UseLogonCredential
$lsass = (Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -ErrorAction SilentlyContinue).RunAsPPL
return ($wdigest -eq 0) -and ($lsass -eq 1)
```

## Apply

```powershell
# Desabilitar WDigest (impede armazenamento de senhas em texto claro)
$wdigestPath = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest"
if (!(Test-Path $wdigestPath)) { New-Item -Path $wdigestPath -Force | Out-Null }
Set-ItemProperty -Path $wdigestPath -Name "UseLogonCredential" -Value 0 -Type DWord -ErrorAction Stop

# Habilitar LSASS como Protected Process Light (RunAsPPL)
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "RunAsPPL" -Value 1 -Type DWord -ErrorAction Stop

Write-Host "    [!] RunAsPPL requer reboot para entrar em vigor." -ForegroundColor Yellow
```

## Rollback

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest" -Name "UseLogonCredential" -Value 1 -Type DWord -ErrorAction SilentlyContinue
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "RunAsPPL" -Value 0 -Type DWord -ErrorAction SilentlyContinue
```

## Remediation Hints

- `RunAsPPL` só é efetivo após reboot — o Check de validação pós-apply deve informar que o status será confirmado no próximo boot.
- Se `Set-ItemProperty` em `Lsa` retornar Access Denied: usar `reg add "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" /v RunAsPPL /t REG_DWORD /d 1 /f`.
- Em sistemas com Secure Boot + UEFI, o RunAsPPL pode ser configurado via Credential Guard (mais robusto). Para ambientes domésticos, o RunAsPPL via registro já é efetivo.
