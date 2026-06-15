# Hardening Doctrine — Windows 10/11
## Version 2.0

> **Changelog v2.0:** Complete architectural refactoring. Topics extracted to `vault/` as individual files. This file is now the doctrine index — lightweight, traceable, and extensible. To add a new topic, create a new file in `vault/` following the standard template and register it here.

---

## How the Agent uses this file

1. The agent reads this index to discover **which topics exist** and in **which file** each is located.
2. For each audit topic, the agent reads the corresponding file in `vault/` and extracts the `Check`, `Apply`, `Rollback`, and `Remediation Hints` blocks.
3. To add a new topic: create the file in the vault following the template and add a row to the table below.

---

## Topic Index

| # | File | Category | Risk (if not applied) |
|---|------|----------|-----------------------|
| 01 | [01-critical-services.md](vault/01-critical-services.md) | Attack Surface Reduction | MEDIUM |
| 02 | [02-smbv1.md](vault/02-smbv1.md) | Legacy Protocols | CRITICAL |
| 03 | [03-windows-update.md](vault/03-windows-update.md) | Patch Management | LOW |
| 04 | [04-password-policy.md](vault/04-password-policy.md) | Access Policy | HIGH |
| 05 | [05-firewall.md](vault/05-firewall.md) | Network Traffic Control | CRITICAL |
| 06 | [06-wsh-block.md](vault/06-wsh-block.md) | Script Execution Control | HIGH |
| 07 | [07-uac-max.md](vault/07-uac-max.md) | Privilege Escalation Control | HIGH |
| 08 | [08-dns-secure.md](vault/08-dns-secure.md) | Privacy and Network | MEDIUM |
| 09 | [09-exploit-protection.md](vault/09-exploit-protection.md) | Memory Protection | HIGH |
| 10 | [10-file-extensions.md](vault/10-file-extensions.md) | Social Engineering Prevention | MEDIUM |
| 11 | [11-powershell-logging.md](vault/11-powershell-logging.md) | Auditing and Intrusion Detection | HIGH |
| 12 | [12-telemetry-block.md](vault/12-telemetry-block.md) | Privacy | LOW/MEDIUM |
| 13 | [13-autorun-block.md](vault/13-autorun-block.md) | Physical Device Security | HIGH |
| 14 | [14-credential-protection.md](vault/14-credential-protection.md) | Credential Dumping Protection | CRITICAL |
| 15 | [15-llmnr-netbios.md](vault/15-llmnr-netbios.md) | Local Network MITM Protection | CRITICAL |
| 16 | [16-rdp-hardening.md](vault/16-rdp-hardening.md) | Remote Access Security | CRITICAL |

---

## Template for New Topics

When creating a new file in `vault/`, use this structure:

```markdown
# Topic XX — [Name]

**Category:** [Category]
**Risk for user:** [NONE/LOW/MEDIUM/HIGH]
**Security risk (if not applied):** [LOW/MEDIUM/HIGH/CRITICAL] — [Vector description]

**Affected Registry Keys:**
- `HKLM\...` → `ValueName`

---

## Check
```powershell
# Return $true if SECURE, $false if VULNERABLE
```

## Apply
```powershell
# Commands to apply hardening
```

## Rollback
```powershell
# Commands to revert the change
```

## Remediation Hints
- Specific remediation hint for this topic.
```
