# Tópico 02 — SMBv1 Protocol

**Categoria:** Protocolos Legados
**Risco para o usuário:** BAIXO — NAS/impressoras com mais de 15 anos podem perder conexão.
**Risco de segurança (não aplicar):** CRÍTICO — Vetor do ataque WannaCry/EternalBlue.

**Chaves de Registro Afetadas:**
- `HKLM\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters` → `SMB1`

---

## Check

```powershell
$feature = (Get-WindowsOptionalFeature -Online -FeatureName "SMB1Protocol" -ErrorAction SilentlyContinue).State
$reg = (Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" -Name "SMB1" -ErrorAction SilentlyContinue).SMB1
return ($feature -eq 'Disabled') -and ($reg -eq 0)
```

## Apply

```powershell
Set-SmbServerConfiguration -EnableSMB1Protocol $false -Force -ErrorAction Stop
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" -Name "SMB1" -Value 0 -Force -ErrorAction Stop
Disable-WindowsOptionalFeature -Online -FeatureName "SMB1Protocol" -NoRestart -WarningAction SilentlyContinue -ErrorAction Stop | Out-Null
```

## Rollback

```powershell
Set-SmbServerConfiguration -EnableSMB1Protocol $true -Force -ErrorAction SilentlyContinue
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" -Name "SMB1" -Value 1 -Force -ErrorAction SilentlyContinue
Enable-WindowsOptionalFeature -Online -FeatureName "SMB1Protocol" -NoRestart -ErrorAction SilentlyContinue | Out-Null
```

## Remediation Hints

- Se `Get-WindowsOptionalFeature` falhar (serviço CBS não disponível): use `dism /online /Disable-Feature /FeatureName:SMB1Protocol /NoRestart` como alternativa.
- Se `Set-SmbServerConfiguration` retornar erro de permissão: forçar via registro é suficiente — o check de registry já valida isso.
- **⚠️ Atenção pós-apply:** `Disable-WindowsOptionalFeature` pode retornar `RestartNeeded = True`. Nesse caso, o Check vai retornar falso mesmo com a configuração correta. Avise o usuário que o status só será confirmado após reboot.
