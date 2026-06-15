# Tópico 01 — Serviços Críticos

**Categoria:** Redução de Superfície de Ataque
**Risco para o usuário:** Xbox App e Fax param de funcionar. Telemetria desativada.
**Risco de segurança (não aplicar):** MÉDIO — RemoteRegistry permite acesso remoto ao registro.

**Chaves de Registro Afetadas:** Nenhuma (serviços apenas)

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

- Se `Set-Service` retornar "Access Denied", tente `sc.exe config [nome] start= disabled` como alternativa.
- Se o serviço não existir (ex: versão mais recente do Windows removeu XboxNetApiSvc), ignore silenciosamente e continue.
