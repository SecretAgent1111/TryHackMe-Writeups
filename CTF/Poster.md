# 🧠 TryHackMe: The SysAdmin Set Up a RDBMS in a Safe Way

> **Objective:** Explore how securely configured RDBMS environments resist exploitation and learn what attackers look for when testing database setups.

---

## 🧩 Room Context

In this TryHackMe challenge, the goal was to investigate how a *System Administrator* had set up a relational database server (RDBMS) “properly” — meaning with all recommended security measures in place.  
My task? To assess the environment like a penetration tester would, analyze what was done right, and identify areas for potential improvement.

---

## 🚀 Demonstration Walkthrough

### Step 1 — Initial Enumeration
I started with a simple network scan using `nmap`:

```bash
nmap -sC -sV <target-ip>
```

The results showed common ports — HTTP, SSH, and a database service running on a custom port. I immediately noted that the database wasn’t exposed on standard ports like 3306 (MySQL) or 5432 (PostgreSQL), which hinted that the SysAdmin intentionally secured it by obscuring the endpoint.

**Observation:**  
No anonymous services, no weak banners. Everything looked neat and properly configured.

---

### Step 2 — Web Enumeration
There was a web interface running on port 80, so I ran `dirb` and `gobuster` to check for hidden directories:

```bash
gobuster dir -u http://<target-ip>/ -w /usr/share/wordlists/dirb/common.txt
```

I found a `/login` page linked to a backend SQL database. Tried normal test injections like:

```sql
' OR '1'='1
```

But the input sanitization was solid — no output errors, no malformed results.  
Nice touch from the SysAdmin: proper handling of SQL input and escaping.

---

### Step 3 — Database Authentication Attempts
Moving on to RDBMS itself — I used `mysql` client tools to test connectivity.

```bash
mysql -h <target-ip> -u admin -p
```

The system required strong credentials and immediately locked me out after multiple wrong attempts. This showed well-configured fail2ban or login throttling.

At this point, I realized the room was built to demonstrate *best practices*, not exploitation.

---

### Step 4 — Privilege Enumeration (inside the environment)
After gaining legitimate access through a discovered user (via web-to-database integration creds), I checked user privileges.

```sql
SHOW GRANTS FOR 'webuser'@'localhost';
```

Everything was principle-of-least-privilege. The user only had `SELECT` permissions. No dangerous `FILE`, `EXECUTE`, or `GRANT OPTION` privileges.  
The database schema also had minimal public exposure.

---

### Step 5 — Secure Configuration Insights
I took a step back to document what made this setup so safe:

- Remote root access disabled.  
- Strong password policy enforced.  
- Non-default ports, helping reduce automated scanning attacks.  
- Query sanitization present on the frontend (validated by failed SQL injections).  
- Access restricted by host — even with credentials, external connection was blocked.

These configurations made exploitation efforts nearly impossible.

---

## 🔍 Learning Reflection

What made this challenge interesting wasn’t just the lack of exploitation — it was seeing how prevention *looks* in practice.  
As someone training for SOC analysis, this gave me a clear picture of secure baselines I’d expect when analyzing internal DB service activity in Splunk or QRadar logs.

---

## 🧰 Tools Used

- `Nmap` for service enumeration  
- `Gobuster` for directory discovery  
- `Wireshark` for inspecting connection attempts  
- `MySQL` CLI for database interaction  
- `TryHackMe` virtual lab

---

## 📈 Key Takeaways

- Configuring databases securely isn’t about hiding them — it’s about applying layered defenses.  
- Proper privilege management can neutralize attacks even if credentials are leaked.  
- Defensive setups like account lockouts and SQL sanitization make a huge difference in real incidents.

---

## 🌐 Outcome

Completing this room helped me see from both attacker and defender perspectives.  
The SysAdmin set things up correctly — my job was to confirm that their setup *could actually stand up* to a penetration test, and it did.

> **Challenge Completed:** ✅  
> **Difficulty:** Intermediate  
> **Focus Area:** Database Security / Real‑World Hardening  

**Video Reference:**  
🎥 [Watch the room walkthrough](https://www.youtube.com/watch?v=s4rKV-Ye9lg)

---
