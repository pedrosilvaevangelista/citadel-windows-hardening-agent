# Tópico 13 — Bloquear AutoRun / AutoPlay

**Categoria:** Segurança de Dispositivos Físicos
**Risco para o usuário:** BAIXO — USBs e CDs precisarão ser abertos manualmente.
**Risco de segurança (não aplicar):** ALTO — AutoRun é vetor clássico de infecção via USB (ex: pendrive infectado).

**Chaves de Registro Afetadas:**
- `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer` → `NoDriveTypeAutoRun`
- `HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer` → `NoDriveTypeAutoRun`
- `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\AutoplayHandlers` → `DisableAutoplay`

---

## Check

```powershell
$val1 = (Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ErrorAction SilentlyContinue).NoDriveTypeAutoRun
$val2 = (Get-ItemProperty "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ErrorAction SilentlyContinue).NoDriveTypeAutoRun
$val3 = (Get-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\AutoplayHandlers" -ErrorAction SilentlyContinue).DisableAutoplay
return ($val1 -eq 255) -and ($val2 -eq 255) -and ($val3 -eq 1)
```

## Apply

```powershell
$policiesHKLM = "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer"
$policiesHKCU = "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer"
$autoplayHKCU = "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\AutoplayHandlers"

foreach ($p in @($policiesHKLM, $policiesHKCU, $autoplayHKCU)) {
    if (!(Test-Path $p)) { New-Item -Path $p -Force | Out-Null }
}

Set-ItemProperty -Path $policiesHKLM -Name "NoDriveTypeAutoRun" -Value 255 -Type DWord -ErrorAction Stop
Set-ItemProperty -Path $policiesHKCU -Name "NoDriveTypeAutoRun" -Value 255 -Type DWord -ErrorAction Stop
Set-ItemProperty -Path $autoplayHKCU -Name "DisableAutoplay" -Value 1 -Type DWord -ErrorAction Stop
```

## Rollback

```powershell
Remove-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer" -Name "NoDriveTypeAutoRun" -ErrorAction SilentlyContinue
Remove-ItemProperty -Path "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer" -Name "NoDriveTypeAutoRun" -ErrorAction SilentlyContinue
Remove-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\AutoplayHandlers" -Name "DisableAutoplay" -ErrorAction SilentlyContinue
```

## Remediation Hints

- Se a chave HKLM falhar por permissão: usar `reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer" /v NoDriveTypeAutoRun /t REG_DWORD /d 255 /f`.
- O valor `255` equivale a `0xFF` — bloqueia todos os tipos de drive (fixo, removível, rede, CD, etc.).
