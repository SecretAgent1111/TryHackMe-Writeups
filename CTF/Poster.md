# TryHackMe: The SysAdmin Set Up a RDBMS in a Safe Way

**Platform:** TryHackMe
**Difficulty:** Intermediate
**Focus Area:** Database Security / Secure RDBMS Configuration
**Category:** Penetration Testing / Defensive Security

---

## Overview

This room was about checking whether a relational database was configured safely. Instead of
exploiting a weak setup, I had to validate that the common attack paths were properly blocked.
That made the room feel more like a real assessment of hardening controls than a normal CTF box.

---

## Task 1: Reconnaissance

### Objective
Identify the exposed services and understand the attack surface.

### What I Did
I started with a full scan to see what was open and whether the database was exposed in a
normal or unusual way.

```bash
nmap -sC -sV -oN initial_scan.txt <target-ip>
```

### Key Findings

- SSH was running on port 22.
- HTTP was running on port 80.
- The database service was on a non-default port, which reduces automated exposure.

### Answer
**What services are exposed on the machine?**
> SSH, HTTP, and a database service on a non-default port

---

## Task 2: Web Enumeration

### Objective
Check whether the web application leaks anything useful.

### What I Did
I brute-forced directories to look for hidden panels or files.

```bash
gobuster dir -u http://<target-ip>/ \
  -w /usr/share/wordlists/dirb/common.txt \
  -o gobuster_results.txt
```

### Key Findings

- I found a login page tied to the database-backed app.
- I did not find exposed admin tools, backup files, or obvious database panels.

### Answer
**What web endpoint was exposed for authentication?**
> /login

---

## Task 3: SQL Injection Testing

### Objective
Test whether the login form was vulnerable to injection.

### What I Did
I tried standard SQL injection payloads against the login form.

```sql
' OR '1'='1' --
' OR 1=1 --
admin'--
" OR ""="
```

### Key Findings

- None of the payloads bypassed authentication.
- The application returned generic errors.
- No stack traces or SQL syntax leaks were visible.

### Answers
**Was the login form vulnerable to SQL injection?**
> No

**What behavior suggested prepared statements were being used?**
> Generic error handling with no query or syntax leakage

---

## Task 4: Database Authentication

### Objective
Check if the database could be accessed directly.

### What I Did
I tested the database login with common accounts.

```bash
mysql -h <target-ip> -u root -p
mysql -h <target-ip> -u admin -p
```

### Key Findings

- Remote root login was blocked.
- Default or blank passwords were not accepted.
- Repeated failed logins appeared to trigger throttling or lockout behavior.

### Answers
**Could you log in remotely as root?**
> No

**What protected the database from brute force attempts?**
> Account lockout / login throttling

---

## Task 5: Privilege Review

### Objective
Check how database privileges were assigned.

### What I Did
After getting legitimate access through the intended path, I reviewed the grants and
user permissions.

```sql
SHOW GRANTS FOR 'webuser'@'localhost';
SELECT user, host, authentication_string FROM mysql.user;
```

### Key Findings

- `root` was limited to `localhost`.
- The application user had only minimal access.
- No dangerous privileges like `FILE`, `DROP`, or `GRANT OPTION` were present.

### Answers
**What privilege model was used for the application account?**
> Least privilege

**Was there any wildcard host account for privileged users?**
> No

---

## Task 6: Security Controls Review

### Objective
Summarize the hardening measures that were in place.

### What I Observed
The setup used multiple layers of protection instead of relying on one control. The database
was hidden from default ports, the web app handled input safely, and the account permissions
were tightly restricted.

### Answer
**What made this RDBMS setup safer than a typical vulnerable lab?**
> Non-default port, restricted root access, prepared statements, generic errors, and least privilege

---

## Tools Used

| Tool      | Purpose                            |
|-----------|------------------------------------|
| Nmap      | Port scanning and service detection |
| Gobuster  | Endpoint enumeration               |
| MySQL CLI | Database authentication testing    |
| TryHackMe | Lab environment                    |

---

## Conclusion

This room showed me that a secure database setup is usually not about one big defense. It is
the combination of proper authentication, limited privileges, safe query handling, and reduced
exposure that makes the difference.
