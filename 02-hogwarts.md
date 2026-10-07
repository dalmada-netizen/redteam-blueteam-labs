# Hogwarts: CTF Penetration Test

**Type:** Red Team · black-box\
**Target:** Linux / Apache + WordPress\
**Methodology:** Full pentest, recon, privilege escalation\
**Result:** Root compromise

## Objective

Compromise the web server, escalate privileges on the underlying Linux host and capture both flags.

## Methodology & attack chain

| Phase | Action | Tools |
|-------|--------|-------|
| 1. Recon | Port scan - only port 80 open (Apache 2.4.46) | Nmap |
| 2. Web enum | Directory fuzzing exposed `/log` and `phpinfo.php` | Gobuster, Dirb |
| 3. Clue decoding | Recovered a Base64-encoded password and a hidden path; decoded a **Brainfuck** payload to reveal a system username | CyberChef |
| 4. CMS enum | Identified WordPress 5.5.3 and user | WPScan, WhatWeb |
| 5. Credential access | Targeted brute force with a themed wordlist found the admin password | WPScan |
| 6. RCE | Injected `system($_GET['cmd'])` into the active theme's `404.php` via the Theme Editor; confirmed execution as `www-data` | WordPress admin |
| 7. Stable shell | Swapped an unstable payload for `php/meterpreter/reverse_tcp` (web-context safe) | msfvenom, Metasploit |
| 8. Lateral move | `su` to the user identified in the Brainfuck clue; read flag 1 (`cat` unavailable → used Python) | — |
| 9. Privilege escalation | Found `/usr/bin/find` with the **SUID bit** set; exploited via GTFOBins (`find . -exec /bin/sh -p \; -quit`) → `euid=0` | GTFOBins |
| 10. Root | Read the final flag from `/root` with root-effective privileges | — |

## Key findings

|   | Vulnerability | Severity |
|---|---------------|----------|
| 1 | RCE via WordPress Theme Editor | Critical |
| 2 | `find` binary with SUID bit | Critical |
| 3 | Weak / reused passwords + secrets in public content | High |
| 4 | `phpinfo.php` exposed | Medium |
| 5 | User in `lxd` group | Medium |

## Remediation

- Set `DISALLOW_FILE_EDIT` in `wp-config.php` and apply least privilege to WordPress roles.
- Remove the SUID bit from non-essential binaries (`chmod u-s /usr/bin/find`) and audit SUID binaries periodically.
- Enforce a strong password policy.
- Remove `phpinfo.php` from production.

## Conclusion

A chain of individually "minor" misconfigurations, an exposed info file, a predictable password, a loose SUID bit combined into full root compromise. Recon and patient enumeration mattered more than any single exploit.
