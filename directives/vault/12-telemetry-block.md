# Tópico 12 — Bloquear Telemetria do OS

**Categoria:** Privacidade
**Risco para o usuário:** BAIXO — Pode impedir participação no Windows Insider Program.
**Risco de segurança (não aplicar):** BAIXO/MÉDIO — Dados de diagnóstico são enviados para a Microsoft.

**Chaves de Registro Afetadas:**
- `HKLM\SOFTWARE\Policies\Microsoft\Windows\DataCollection` → `AllowTelemetry`

---

## Check

```powershell
$val = (Get-ItemProperty "HKLM:\SOFTWARE\Policies\Microsoft\Windows\DataCollection" -ErrorAction SilentlyContinue).AllowTelemetry
return $val -eq 0
```

## Apply

```powershell
$path = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\DataCollection"
if (!(Test-Path $path)) { New-Item -Path $path -Force | Out-Null }
Set-ItemProperty -Path $path -Name "AllowTelemetry" -Value 0 -ErrorAction Stop

# Bloquear também via serviço DiagTrack (complementar ao Tópico 1)
Stop-Service -Name "DiagTrack" -Force -ErrorAction SilentlyContinue
Set-Service -Name "DiagTrack" -StartupType Disabled -ErrorAction SilentlyContinue
```

## Rollback

```powershell
Remove-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\DataCollection" -Name "AllowTelemetry" -ErrorAction SilentlyContinue
Set-Service -Name "DiagTrack" -StartupType Automatic -ErrorAction SilentlyContinue
```

## Remediation Hints

- Em edições **Windows 10 Home**, o valor `0` pode ser ignorado pelo sistema — o mínimo efetivo é `1` (Básico). Se o Check sempre retornar falso mesmo após o Apply, avise o usuário que a edição Home limita esse controle.
