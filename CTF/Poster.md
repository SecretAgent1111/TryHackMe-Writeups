# TryHackMe: The SysAdmin Set Up a RDBMS in a Safe Way

**Platform:** TryHackMe


**Difficulty:** Intermediate


**Focus Area:** Database Security / Secure RDBMS Configuration


**Category:** Penetration Testing / Defensive Security

---

## Overview

This room simulates a real-world scenario where a System Administrator has configured a Relational Database Management System (RDBMS) following industry security best practices. Rather than finding and exploiting vulnerabilities, the objective is to assess the environment from an attacker's perspective — confirming that the defenses hold up under a structured penetration test.

This write-up documents my methodology, observations, and the security controls I encountered during the assessment.

---

## Objectives

- Perform black-box enumeration of a hardened database environment
- Identify and validate security controls put in place by the SysAdmin
- Attempt common database attack vectors and document why they fail
- Derive actionable takeaways applicable to real-world database hardening

---

## Methodology

The assessment followed a structured penetration testing approach aligned with common frameworks:

1. Reconnaissance and port enumeration
2. Web application enumeration and injection testing
3. Database authentication testing
4. Privilege review after legitimate access
5. Configuration analysis and documentation

---

## Walkthrough

### Phase 1 — Network Reconnaissance

Initial enumeration was performed using `nmap` with service detection and default scripts enabled:
```bash
nmap -sC -sV -oN initial_scan.txt <target-ip>
```

**Findings:**

- SSH running on port 22 — standard, no anonymous access
- HTTP service on port 80 — web application present
- Database service running on a **non-default port** — not 3306 (MySQL) or 5432 (PostgreSQL)

Running the database on a non-standard port is a basic but effective measure against automated scanners and opportunistic bots that specifically probe default database ports. While this is not a substitute for authentication controls, it reduces noise and lowers the attack surface from unsophisticated threats.

No service banners leaked version information. This suggests the SysAdmin had configured banner suppression, which prevents attackers from fingerprinting exact software versions and targeting known CVEs.

---

### Phase 2 — Web Application Enumeration

A web interface was accessible on port 80. Directory brute-forcing was performed using Gobuster:
```bash
gobuster dir -u http://<target-ip>/ -w /usr/share/wordlists/dirb/common.txt -o gobuster_results.txt
```

**Discovered endpoints:**

- `/login` — Authentication page connected to the backend database
- No exposed `/admin`, `/phpmyadmin`, `/db`, or backup file paths found

#### SQL Injection Testing

The login form was tested for common SQL injection payloads:
```sql
' OR '1'='1' --
' OR 1=1 --
admin'--
" OR ""="
```

None of the payloads produced errors, unexpected behavior, or authentication bypass. The application returned generic error messages without disclosing stack traces or query structure — a sign of proper error handling and input sanitization.

This behavior is consistent with the use of **prepared statements or parameterized queries**, which separate SQL logic from user-supplied input entirely, making classic injection attacks ineffective regardless of payload complexity.

---

### Phase 3 — Database Authentication Testing

Direct connection to the database service was attempted using the MySQL CLI:
```bash
mysql -h <target-ip> -u root -p
mysql -h <target-ip> -u admin -p
```

**Observations:**

- Remote root login was refused outright — `root` was restricted to `localhost` only
- Multiple failed authentication attempts triggered an automatic lockout, consistent with either `fail2ban` or MySQL's built-in `max_connect_errors` threshold
- No default or blank-password accounts were accessible

This phase confirmed that the most common database attack vectors — default credentials, brute force, and remote root access — were all effectively mitigated.

---

### Phase 4 — Privilege Enumeration (Post Legitimate Access)

After obtaining credentials through authorized means (web application integration credentials discovered in the room's intended path), internal access was used to review the privilege model:
```sql
SHOW GRANTS FOR 'webuser'@'localhost';
SELECT user, host, authentication_string FROM mysql.user;
```

**Privilege audit results:**

| User      | Host        | Privileges Granted         |
|-----------|-------------|----------------------------|
| root      | localhost   | ALL (local only)           |
| webuser   | localhost   | SELECT on app database     |
| No wildcard host (`%`) accounts found |

The `webuser` account — the one used by the web application to query the database — had only `SELECT` privileges scoped to a single database. There were no `FILE`, `EXECUTE`, `DROP`, `CREATE`, or `GRANT OPTION` privileges assigned.

This is a textbook implementation of the **Principle of Least Privilege (PoLP)**. Even in a scenario where the web application credentials were compromised, an attacker would be unable to write files, modify data, escalate privileges, or pivot further into the system.

---

### Phase 5 — Configuration Review

A final review was conducted to catalogue all observed security controls:

**Authentication and Access Controls:**
- Remote root login disabled (`bind-address` restricted, root limited to `localhost`)
- Account lockout policy active after repeated failed login attempts
- No wildcard host bindings on privileged accounts
- Strong password policy enforced at the database level

**Network-Level Hardening:**
- Database not listening on default ports, reducing automated scanning exposure
- Firewall rules limiting database port access to the application server only (external connections blocked even with valid credentials)

**Application-Level Security:**
- Parameterized queries / prepared statements used throughout the web application
- Generic error messages returned to the client — no query or stack trace leakage
- No exposed database administration interfaces (phpMyAdmin, Adminer, etc.)

**Privilege Management:**
- Least privilege enforced on all application-facing accounts
- Schema access scoped tightly — no unnecessary table or database exposure
- No accounts with dangerous privilege combinations (`FILE` + write access, etc.)

---

## Security Controls Summary

| Control                          | Status       | Impact                                      |
|----------------------------------|--------------|---------------------------------------------|
| Remote root login disabled       | Implemented  | Prevents direct root compromise remotely    |
| Non-default database port        | Implemented  | Reduces automated scan exposure             |
| Account lockout / throttling     | Implemented  | Mitigates brute force attacks               |
| Parameterized queries            | Implemented  | Eliminates SQL injection vectors            |
| Principle of least privilege     | Implemented  | Limits blast radius of credential theft     |
| Host-based access restriction    | Implemented  | Blocks unauthorized remote DB connections   |
| Banner suppression               | Implemented  | Prevents version-based fingerprinting       |
| No exposed admin interfaces      | Implemented  | Reduces web-facing attack surface           |

---

## Tools Used

| Tool        | Purpose                                      |
|-------------|----------------------------------------------|
| Nmap        | Port scanning and service fingerprinting     |
| Gobuster    | Web directory and endpoint enumeration       |
| MySQL CLI   | Direct database connection and query testing |
| Wireshark   | Packet inspection during connection attempts |
| TryHackMe   | Controlled lab environment                   |

---

