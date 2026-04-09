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

```
?id=1
?id=2
/view?file=../admin.txt
/view?file=../../etc/passwd
```

After a bit of trial and error, I found a working vector by manipulating how the app handles
one of its parameters or inputs.

### Answer
**What vulnerability was mainly being tested in this phase?**
> Input manipulation / data access vulnerability (likely IDOR or path traversal)

---

## Task 4: Exploitation

### Objective
Use the found vector to get unauthorized data.

### What I Did
Once I had a working payload, I used `curl` to test it:

```bash
curl "http://<target-ip>/vulnerable-endpoint"
```

That gave me access to something that was supposed to be hidden or restricted. I also tried
a few more requests directly:

```bash
curl http://<target-ip>/admin.php
curl http://<target-ip>/config.php
```

Some of these returned useful info because the app wasn't properly restricting access.

### Answer
**Through what endpoint did you first get sensitive data access?**
> The vulnerable endpoint discovered during enumeration and testing

---

## Task 5: Flag Retrieval

### Objective
Find and submit the flag.

### What I Did
After finding the right path, I checked the hidden directory and file I had discovered earlier.

```bash
curl http://<target-ip>/flag.txt
```

The flag was in `THM{...}` format.

### Answer
**What is the flag format used in the room?**
> `THM{...}`

---

## Tools & Techniques Summary

| Task | Tool | Purpose |
|------|------|---------|
| Port Scanning | Nmap | Map exposed services |
| Directory Brute-forcing | Gobuster | Find hidden pages and files |
| Traffic Inspection | Burp Suite | Inspect and manipulate HTTP requests |
| Direct Testing | cURL | Quick endpoint checks |
| Frontend Inspection | Firefox DevTools | Check client-side code and structure |
