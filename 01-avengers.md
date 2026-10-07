# La Iniciativa Vengadores: Web Application Penetration Test

**Type:** Red Team · Black-box web pentest\
**Target:** Linux / Apache web application\
**Methodology:** OWASP Testing Guide v4 · MITRE ATT&CK\
**Result:** Full system compromise (RCE)

## Objective

Assess a web application from the perspective of an external attacker with no prior credentials, identify vulnerabilities and demonstrate their real-world impact.

## Methodology & attack chain

| Phase | Action | Tools |
|-------|--------|-------|
| 1. Recon | Full port scan (`-p- -sV`); identified name-based virtual hosting | Nmap |
| 2. Enumeration | Source code revealed a search field and a hidden upload form | Manual review |
| 3. Blind SQLi | Confirmed injection via boolean (`' OR '1'='1`) and time-based tests; dumped the database | Burp Suite, sqlmap |
| 4. Credential access | Extracted an unsalted MD5 password hash, cracked automatically | sqlmap |
| 5. Admin access | Authenticated to the admin panel with the recovered credentials |  |
| 6. Webshell upload | Reopened the hidden upload endpoint and uploaded a PHP reverse shell | Burp Suite |
| 7. RCE | Caught the reverse shell as `www-data` and captured the final flag | Netcat |

## Key findings

|   | Vulnerability | Severity |
|---|---------------|----------|
| 1 | Blind SQL Injection (`searchVal`) | Critical |
| 2 | Unrestricted File Upload | Critical |
| 3 | Sensitive info in source code | High |
| 4 | Weak password storage (unsalted MD5) | High |

## Remediation

- **Critical** - Use parameterized queries on every SQL query; apply least privilege to the DB user.
- **Critical** - Validate the file extension and MIME type on the server for all uploads, rename uploads, store them outside the web root and disable PHP execution in `/uploads`.
- Migrate password storage to bcrypt or Argon2id using PHP's native functions: `password_hash()` and `password_verify()`.
- Remove all commented code from the frontend before deployment, and add a code review step before production.

**Additional hardening**
- Update Apache and OpenSSH to current, patched versions.
- Disable directory listing in Apache (`Options -Indexes`).
- Enforce a strong password policy: minimum 12 characters, mixing uppercase, lowercase, numbers and symbols.

## Conclusion

Two independent critical flaws (SQLi + file upload) reinforced each other; one supplied the credentials, the other the execution vector. Neither would have been as devastating alone.
