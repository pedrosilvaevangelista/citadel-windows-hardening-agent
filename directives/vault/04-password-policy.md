# Tópico 04 — Hardening de Senha

**Categoria:** Política de Acesso
**Risco para o usuário:** ALTO — Lockout de 15 min após 5 tentativas erradas. Senhas complexas obrigatórias.
**Risco de segurança (não aplicar):** ALTO — Senhas fracas são o vetor #1 de comprometimento.

**Chaves de Registro Afetadas:** Nenhuma diretamente (usa `secedit` para política de segurança local)

---

## Check

```powershell
$tempFile = "$env:TEMP\secpol_check_$(Get-Random).cfg"
secedit /export /cfg "$tempFile" /quiet
$content = Get-Content $tempFile -ErrorAction SilentlyContinue
Remove-Item $tempFile -Force -ErrorAction SilentlyContinue
$minLen = ($content | Select-String "MinimumPasswordLength\s*=\s*(\d+)").Matches.Groups[1].Value
$complexity = ($content | Select-String "PasswordComplexity\s*=\s*(\d+)").Matches.Groups[1].Value
return ([int]$minLen -ge 12) -and ([int]$complexity -eq 1)
```

## Apply

```powershell
$secpolContent = @"
[Unicode]
Unicode=yes
[System Access]
MinimumPasswordLength = 12
PasswordComplexity = 1
PasswordHistorySize = 24
MaximumPasswordAge = 90
MinimumPasswordAge = 1
LockoutBadCount = 5
ResetLockoutCount = 15
LockoutDuration = 15
[Version]
signature="`$CHICAGO`$"
Revision=1
"@
$secpolFile = [System.IO.Path]::Combine([System.IO.Path]::GetTempPath(), [System.IO.Path]::GetRandomFileName() + '.inf')
$secpolDb = [System.IO.Path]::Combine([System.IO.Path]::GetTempPath(), [System.IO.Path]::GetRandomFileName() + '.sdb')
$secpolContent | Out-File -Encoding Unicode -FilePath $secpolFile
secedit /configure /db "$secpolDb" /cfg "$secpolFile" /areas SECURITYPOLICY /quiet
Remove-Item $secpolFile -Force -ErrorAction SilentlyContinue
Remove-Item $secpolDb -Force -ErrorAction SilentlyContinue
gpupdate /force | Out-Null
```

## Rollback

```powershell
# Restaurar política padrão do Windows (valores mínimos)
net accounts /minpwlen:0 /maxpwage:unlimited /minpwage:0 /uniquepw:0 /lockoutthreshold:0
```

## Remediation Hints

- Se `secedit /configure` falhar com erro de encoding: re-salvar o arquivo `.inf` com `Out-File -Encoding Default` (ANSI).
- Se `gpupdate /force` travar: executar `gpupdate /force /boot` ou simplesmente ignorar — a política secedit já foi aplicada.
- Alternativa direta: `net accounts /minpwlen:12 /lockoutthreshold:5 /lockoutduration:15 /lockoutwindow:15`.
- **⚠️ Caminho com espaços:** `$env:TEMP` pode conter espaços e quebrar o `secedit`. Sempre use `[System.IO.Path]::Combine(...)` para gerar o path seguro (já feito no Apply acima).
