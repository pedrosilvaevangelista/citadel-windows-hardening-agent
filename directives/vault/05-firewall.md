# Tópico 05 — Firewall (Todos os Perfis)

**Categoria:** Controle de Tráfego de Rede
**Risco para o usuário:** BAIXO — Pode bloquear apps locais sem regra configurada.
**Risco de segurança (não aplicar):** CRÍTICO — Firewall desativado expõe portas a toda a rede.

**Chaves de Registro Afetadas:** Nenhuma (gerenciado por cmdlets de firewall)

---

## Check

```powershell
$enabled = (Get-NetFirewallProfile -Profile Domain,Private,Public | Where-Object {$_.Enabled -eq 'True'}).Count
return $enabled -eq 3
```

## Apply

```powershell
Set-NetFirewallProfile -Profile Domain,Private,Public -Enabled True -ErrorAction Stop
```

## Rollback

```powershell
Set-NetFirewallProfile -Profile Domain,Private,Public -Enabled False -ErrorAction SilentlyContinue
```

## Remediation Hints

- Se `Set-NetFirewallProfile` falhar: use `netsh advfirewall set allprofiles state on` como alternativa.
