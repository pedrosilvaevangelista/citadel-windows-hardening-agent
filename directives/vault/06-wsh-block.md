# Tópico 06 — Bloquear WSH (.VBS / .JS)

**Categoria:** Controle de Execução de Scripts
**Risco para o usuário:** ALTO — Instaladores e scripts de automação legados em VBScript ou JScript pararão de funcionar.
**Risco de segurança (não aplicar):** ALTO — WSH é vetor clássico de malware via e-mail (ex: .vbs disfarçado).

**Chaves de Registro Afetadas:**
- `HKLM\SOFTWARE\Microsoft\Windows Script Host\Settings` → `Enabled`

---

## Check

```powershell
$val = (Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows Script Host\Settings" -ErrorAction SilentlyContinue).Enabled
return $val -eq 0
```

## Apply

```powershell
$path = "HKLM:\SOFTWARE\Microsoft\Windows Script Host\Settings"
if (!(Test-Path $path)) { New-Item -Path $path -Force | Out-Null }
Set-ItemProperty -Path $path -Name "Enabled" -Value 0 -ErrorAction Stop
```

## Rollback

```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows Script Host\Settings" -Name "Enabled" -Value 1 -ErrorAction SilentlyContinue
```

## Remediation Hints

- Se a chave não existir e `New-Item` falhar: usar `reg add "HKLM\SOFTWARE\Microsoft\Windows Script Host\Settings" /v Enabled /t REG_DWORD /d 0 /f`.
