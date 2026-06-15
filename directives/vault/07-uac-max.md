# Tópico 07 — UAC Nível Máximo

**Categoria:** Controle de Privilege Escalation
**Risco para o usuário:** BAIXO — Mais prompts de permissão para ações administrativas.
**Risco de segurança (não aplicar):** ALTO — UAC rebaixado permite elevação silenciosa de privilégio.

**Chaves de Registro Afetadas:**
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

- Esta chave raramente falha pois sempre existe. Se falhar, provavelmente é problema de privilégio — verificar se o terminal está como Admin.
