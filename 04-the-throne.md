# The Throne: Active Directory Red Team Assessment

**Type:** Red Team · Grey-box internal assessment\
**Target:** Active Directory environment | perimeter Linux host + internal Windows domain\
**Methodology:** Cyber Kill Chain · MITRE ATT&CK\
**Result:** Full domain compromise (Domain Admin / Golden Ticket)


## Objective

Determine whether an external attacker, starting from unprivileged access to a single exposed Linux service, could escalate all the way to full administrative control of the Windows domain. **Outcome: yes** achieved through chained configuration weaknesses, with no 0-days or unpatched software.

## Attack chain

| Phase | Action | Tools |
|-------|--------|-------|
| 1. Initial access | Anonymous **write** access to FTP root = web root → uploaded webshell, executed over HTTP as `www-data` | Nmap, FTP |
| 2. Privilege escalation | Found a local script leaking an MD5 hash + a **plaintext domain credential**; cracked the hash offline | Hashcat |
| 3. Pivoting | Host with two network interfaces and no segmentation → built a reverse **SOCKS tunnel** into the internal network | chisel, Proxychains |
| 4. AD enum + Kerberoasting | Enumerated SPNs; a service account (`iis_service`) was a **Domain Admin**; requested its TGS and cracked it offline | PowerView, Impacket |
| 5. DCSync | With Domain Admin, replicated the directory to dump all NTLM hashes, including **krbtgt** | impacket-secretsdump |
| 6. Golden Ticket | Forged a Kerberos ticket from the krbtgt hash + domain SID → SYSTEM on the DC, persistent control | impacket-ticketer |

## Key findings 

| ID  | Vulnerability | Severity |
|-----|---------------|----------|
| V01 | Anonymous write access on FTP | Critical |
| V02 | FTP directory executable via HTTP (RCE) | Critical |
| V03 | Domain credentials in plaintext | Critical |
| V04 | No network segmentation | High |
| V05 | Service account in Domain Admins | Critical |
| V06 | Weak service-account password (Kerberoastable) | Critical |
| V07 | DCSync + Golden Ticket | Critical |
| V08 | SMB null session on the DC | Medium |


## Priority remediation

- Disable anonymous FTP write; **never** place a writable directory inside a web root with an active interpreter.
- Remove plaintext credentials from scripts; use a secrets vault or **gMSA**.
- **No service account should belong to Domain Admins**, apply least privilege and long random passwords (25+ chars) or gMSA.
- Enforce network segmentation between perimeter and internal AD zones.
- Rotate the **krbtgt** password (twice) on any suspicion of compromise; monitor DCSync and anomalous Kerberos requests.

## Conclusion

Several misconfigurations of moderate to high severity, chained together, produced total domain compromise with no software exploit. Fixing any single V01, V03 or V05, would have broken the whole chain. AD security is less about one isolated control and more about eliminating cumulative escalation paths.
