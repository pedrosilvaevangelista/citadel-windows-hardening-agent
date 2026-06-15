# Hardening Doctrine — Windows 10/11
## Versão 2.0

> **Changelog v2.0:** Refatoração arquitetural completa. Tópicos extraídos para `vault/` como arquivos individuais. Este arquivo agora é o índice da doutrina — leve, rastreável e extensível. Para adicionar um novo tópico, crie um novo arquivo em `vault/` seguindo o template padrão e registre-o aqui.

---

## Como o Agente usa este arquivo

1. O agente lê este índice para descobrir **quais tópicos existem** e em **qual arquivo** cada um está.
2. Para cada tópico da auditoria, o agente lê o arquivo correspondente em `vault/` e extrai os blocos `Check`, `Apply`, `Rollback` e `Remediation Hints`.
3. Para adicionar um novo tópico: crie o arquivo na vault seguindo o template e adicione uma linha na tabela abaixo.

---

## Índice de Tópicos

| # | Arquivo | Categoria | Risco (não aplicar) |
|---|---------|-----------|---------------------|
| 01 | [01-critical-services.md](vault/01-critical-services.md) | Redução de Superfície de Ataque | MÉDIO |
| 02 | [02-smbv1.md](vault/02-smbv1.md) | Protocolos Legados | CRÍTICO |
| 03 | [03-windows-update.md](vault/03-windows-update.md) | Gerenciamento de Patches | BAIXO |
| 04 | [04-password-policy.md](vault/04-password-policy.md) | Política de Acesso | ALTO |
| 05 | [05-firewall.md](vault/05-firewall.md) | Controle de Tráfego de Rede | CRÍTICO |
| 06 | [06-wsh-block.md](vault/06-wsh-block.md) | Controle de Execução de Scripts | ALTO |
| 07 | [07-uac-max.md](vault/07-uac-max.md) | Controle de Privilege Escalation | ALTO |
| 08 | [08-dns-secure.md](vault/08-dns-secure.md) | Privacidade e Rede | MÉDIO |
| 09 | [09-exploit-protection.md](vault/09-exploit-protection.md) | Proteção de Memória | ALTO |
| 10 | [10-file-extensions.md](vault/10-file-extensions.md) | Prevenção de Engenharia Social | MÉDIO |
| 11 | [11-powershell-logging.md](vault/11-powershell-logging.md) | Auditoria e Detecção de Intrusão | ALTO |
| 12 | [12-telemetry-block.md](vault/12-telemetry-block.md) | Privacidade | BAIXO/MÉDIO |
| 13 | [13-autorun-block.md](vault/13-autorun-block.md) | Segurança de Dispositivos Físicos | ALTO |
| 14 | [14-credential-protection.md](vault/14-credential-protection.md) | Proteção Contra Credential Dumping | CRÍTICO |
| 15 | [15-llmnr-netbios.md](vault/15-llmnr-netbios.md) | Proteção Contra MITM em Rede Local | CRÍTICO |
| 16 | [16-rdp-hardening.md](vault/16-rdp-hardening.md) | Segurança de Acesso Remoto | CRÍTICO |

---

## Template para Novos Tópicos

Ao criar um novo arquivo em `vault/`, use esta estrutura:

```markdown
# Tópico XX — [Nome]

**Categoria:** [Categoria]
**Risco para o usuário:** [NENHUM/BAIXO/MÉDIO/ALTO]
**Risco de segurança (não aplicar):** [BAIXO/MÉDIO/ALTO/CRÍTICO] — [Descrição do vetor]

**Chaves de Registro Afetadas:**
- `HKLM\...` → `ValorNome`

---

## Check
```powershell
# Retornar $true se SEGURO, $false se VULNERÁVEL
```

## Apply
```powershell
# Comandos para aplicar o hardening
```

## Rollback
```powershell
# Comandos para reverter a mudança
```

## Remediation Hints
- Dica de remediação específica para este tópico.
```
