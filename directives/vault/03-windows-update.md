# Tópico 03 — Controle de Windows Update

**Categoria:** Gerenciamento de Patches
**Risco para o usuário:** MÉDIO — Requer disciplina para instalar atualizações manualmente.
**Risco de segurança (não aplicar):** BAIXO — Auto-reboot pode causar perda de dados em trabalho crítico.

**Chaves de Registro Afetadas:**
- `HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU` → `AUOptions`, `NoAutoRebootWithLoggedOnUsers`

---

## Check

```powershell
$val = (Get-ItemProperty "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" -ErrorAction SilentlyContinue).AUOptions
return $val -eq 2
```

## Apply

```powershell
$path = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU"
if (!(Test-Path $path)) { New-Item -Path $path -Force | Out-Null }
Set-ItemProperty -Path $path -Name "AUOptions" -Value 2 -ErrorAction Stop
Set-ItemProperty -Path $path -Name "NoAutoRebootWithLoggedOnUsers" -Value 1 -ErrorAction Stop
```

## Rollback

```powershell
$path = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU"
Remove-ItemProperty -Path $path -Name "AUOptions" -ErrorAction SilentlyContinue
Remove-ItemProperty -Path $path -Name "NoAutoRebootWithLoggedOnUsers" -ErrorAction SilentlyContinue
```

## Remediation Hints

- Se o caminho de registro não puder ser criado: verifique permissões em `HKLM:\SOFTWARE\Policies\Microsoft\Windows`. Tente `takeown` na chave pai.
- Em máquinas de domínio, GPO do servidor pode sobrescrever esse valor. Avise o usuário que a configuração pode não persistir.
