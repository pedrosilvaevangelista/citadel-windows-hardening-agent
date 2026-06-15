# Tópico 08 — DNS Seguro (1.1.1.1 Cloudflare)

**Categoria:** Privacidade e Rede
**Risco para o usuário:** MÉDIO — Se Cloudflare cair, a internet para. Em redes corporativas, quebra resolução de domínios internos.
**Risco de segurança (não aplicar):** MÉDIO — DNS padrão do ISP pode ser interceptado (DNS Spoofing).

**Chaves de Registro Afetadas:** Nenhuma (gerenciado por cmdlets de rede)

> **⚠️ SKIP AUTOMÁTICO EM DOMÍNIO**: Se o Pre-Flight detectar que a máquina está em Active Directory (`PartOfDomain = True`), este tópico deve ser marcado como `SKIP_DOMAIN` e não deve ser aplicado sem confirmação explícita do usuário.

---

## Check

```powershell
$addresses = (Get-DnsClientServerAddress -AddressFamily IPv4 -ErrorAction SilentlyContinue).ServerAddresses
return $addresses -contains "1.1.1.1"
```

## Apply

```powershell
Get-NetAdapter | Where-Object Status -eq 'Up' | Set-DnsClientServerAddress -ServerAddresses ("1.1.1.1", "1.0.0.1") -ErrorAction Stop
```

## Rollback

```powershell
Get-NetAdapter | Where-Object Status -eq 'Up' | Set-DnsClientServerAddress -ResetServerAddresses -ErrorAction SilentlyContinue
```

## Remediation Hints

- Se `Set-DnsClientServerAddress` falhar: usar `netsh interface ip set dns "[nome_adapter]" static 1.1.1.1` para cada adaptador ativo.
- Para obter os nomes dos adaptadores: `Get-NetAdapter | Where-Object Status -eq 'Up' | Select-Object Name`.
