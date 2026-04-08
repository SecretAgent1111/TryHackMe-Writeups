# Valentine — TryHackMe Writeup

## Room Overview

Valentine is an entry-level web room from the Love at First Breach 2026 event. The target is
a simple web app, likely themed around dating/matchmaking. The goal is to enumerate the app,
find a weak input or parameter handling issue, and use it to access something that should not
be exposed.

| Field | Info |
|-------|------|
| Difficulty | Entry-Level |
| Category | Web Security |
| Tools | Nmap, Gobuster, Burp Suite, cURL, Firefox DevTools |

---

## Task 1: Reconnaissance

### Objective
Find what services are exposed and map the web app surface.

### What I Did
I started the machine and grabbed the target IP. Then I did a quick ping check:

```bash
ping <target-ip>
```

Machine was up and responding, so I moved on to scanning.

```bash
nmap -sV -sC <target-ip>
```

This showed that port 80 (HTTP) was open. Nothing else obvious stood out, so I focused on
the web app. Then I ran directory enumeration:

```bash
gobuster dir -u http://<target-ip> \
  -w /usr/share/wordlists/dirb/common.txt \
  -x php,txt,html
```

This gave me a few hidden endpoints and files I needed to check later.

### Answer
**What port is the web application running on?**
> 80

---

## Task 2: Application Exploration

### Objective
Understand how the app behaves and where user input is used.

### What I Did
I opened the site in Firefox and clicked around: login and registration pages, profile pages,
and any search or input fields. I also fired up Burp Suite and sent some requests through it
to see what parameters were being passed and how the app responded.

### Answer
**What type of application is Valentine themed around?**
> A dating/matchmaking site

---

## Task 3: Vulnerability Testing

### Objective
Check for common web vulnerabilities.

### What I Did
I tested a few standard attack paths.

XSS:

```javascript
<script>alert(1)</script>
<img src=x onerror=alert(1)>
```

SQL injection:

```sql
' OR '1'='1
' OR 1=1--
admin'--
```

IDOR / path traversal:
