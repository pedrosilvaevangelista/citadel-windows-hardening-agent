# Topic 07 — Max UAC Level

**Category:** Privilege Escalation Control
**Risk for user:** LOW — More permission prompts for administrative actions.
**Security risk (if not applied):** HIGH — Lowered UAC allows silent privilege elevation.

**Affected Registry Keys:**
- `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System` → `ConsentPromptBehaviorAdmin`, `PromptOnSecureDesktop`

---

## Check

```powershell
$val = (Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -ErrorAction SilentlyContinue).ConsentPromptBehaviorAdmin
return $val -eq 2
```

## Apply

```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "ConsentPromptBehaviorAdmin" -Value 2 -ErrorAction Stop
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "PromptOnSecureDesktop" -Value 1 -ErrorAction Stop
```

## Rollback

```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "ConsentPromptBehaviorAdmin" -Value 5 -ErrorAction SilentlyContinue
```

## Remediation Hints

- This key rarely fails because it always exists. If it fails, it is probably a privilege issue — verify if the terminal is running as Admin.
