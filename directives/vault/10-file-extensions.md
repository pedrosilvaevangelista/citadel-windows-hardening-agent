# Topic 10 — Show File Extensions

**Category:** Social Engineering Prevention
**Risk for user:** NONE — Visual change only.
**Security risk (if not applied):** MEDIUM — User may open `document.pdf.exe` thinking it is a PDF.

**Affected Registry Keys:**
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

- The `Stop-Process -Name explorer` will close and briefly reopen the taskbar — this is expected and necessary to apply the visual change.
- If Explorer does not restart on its own: manually run `Start-Process explorer`.
