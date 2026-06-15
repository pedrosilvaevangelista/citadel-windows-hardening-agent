# Tópico 10 — Mostrar Extensões de Arquivo

**Categoria:** Prevenção de Engenharia Social
**Risco para o usuário:** NENHUM — Apenas mudança visual.
**Risco de segurança (não aplicar):** MÉDIO — Usuário pode abrir `documento.pdf.exe` pensando ser um PDF.

**Chaves de Registro Afetadas:**
- `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced` → `HideFileExt`

---

## Check

```powershell
$val = (Get-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced" -ErrorAction SilentlyContinue).HideFileExt
return $val -eq 0
```

## Apply

```powershell
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced" -Name "HideFileExt" -Value 0 -ErrorAction Stop
Stop-Process -Name explorer -Force -ErrorAction SilentlyContinue
```

## Rollback

```powershell
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced" -Name "HideFileExt" -Value 1 -ErrorAction SilentlyContinue
Stop-Process -Name explorer -Force -ErrorAction SilentlyContinue
```

## Remediation Hints

- O `Stop-Process -Name explorer` vai fechar e reabrir a barra de tarefas brevemente — isso é esperado e necessário para aplicar a mudança visual.
- Se o Explorer não reiniciar sozinho: executar `Start-Process explorer` manualmente.
