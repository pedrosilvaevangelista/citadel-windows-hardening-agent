# Tópico 11 — PowerShell Advanced Logging

**Categoria:** Auditoria e Detecção de Intrusão
**Risco para o usuário:** BAIXO — Aumenta tamanho dos logs no Event Viewer.
**Risco de segurança (não aplicar):** ALTO — Sem logging, atividade maliciosa de PowerShell é invisível.

**Chaves de Registro Afetadas:**
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

- Se a criação de `New-Item` falhar em algum subpath: usar `reg add` para cada chave individualmente.
- O nome do valor `"*"` em `ModuleNames` pode causar conflito com wildcards do PowerShell. Usar `New-ItemProperty` (já feito no Apply) em vez de `Set-ItemProperty`.
