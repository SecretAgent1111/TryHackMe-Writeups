# Valentine - TryHackMe Writeup

Just finished the Valentine room from the Love at First Breach 2026 event. Pretty straightforward challenge but definitely a good warm-up.

## Room Details
- **Name:** Valentine
- **Difficulty:** Entry-Level
- **Category:** Web Security
- **Platform:** TryHackMe

---

## Initial Setup

Started the machine and got the target IP. First thing I did was ping it to make sure everything was up and running.

```bash
ping <target-ip>
```

Got responses back, so we're good to go.

---

## Reconnaissance

Opened up the web app in Firefox to see what we're dealing with. Looks like some kind of dating/matchmaking site - fitting for the Valentine theme I guess.

Ran a quick nmap scan to see what ports are open:

```bash
nmap -sV -sC <target-ip>
```

Found port 80 open running HTTP. Nothing too surprising there.

Decided to run gobuster to find any hidden directories:

```bash
gobuster dir -u http://<target-ip> -w /usr/share/wordlists/dirb/common.txt -x php,txt,html
```

This found a few interesting endpoints that I made note of.

---

## Exploring the Application

Spent some time clicking around the site. Checked out:
- Login forms
- Registration page
- Profile pages
- Any search or input fields

Opened Burp Suite to intercept requests and see what's happening under the hood. Found some interesting parameters being passed around.

---

## Finding the Vulnerability

Started testing the usual suspects:

**XSS attempts:**
```javascript
<script>alert(1)</script>
<img src=x onerror=alert(1)>
```

**SQL injection:**
```sql
' OR '1'='1
' OR 1=1--
admin'--
```

**IDOR testing:**
Tried manipulating URL parameters like `?id=1`, `?id=2`, etc. to see if I could access other users' data.

Also tested for path traversal:
```
/view?file=../admin.txt
/view?file=../../etc/passwd
```

After some trial and error, found a working exploit vector. Not going to spoil exactly what it was, but it involved manipulating how the application handles user input.

---

## Exploitation

Once I found the vulnerability, used curl to test it out:

```bash
curl "http://<target-ip>/vulnerable-endpoint"
```

Bingo. Got access to some sensitive data that wasn't supposed to be publicly accessible.

Tried a few variations:
```bash
curl http://<target-ip>/admin.php
curl http://<target-ip>/config.php
```

---

## Getting the Flag

After poking around the accessible files, found the flag. It was in one of the hidden directories I discovered earlier.

```bash
curl http://<target-ip>/flag.txt
```

Flag format: `THM{...}`

Submitted it on TryHackMe and got the points.

---

## Tools I Used

- **Nmap** - Port scanning
- **GoBuster** - Directory enumeration  
- **Burp Suite** - Intercepting HTTP traffic
- **cURL** - Making HTTP requests
- **Firefox DevTools** - Inspecting frontend code

---

## What I Learned

This room was a good refresher on:
- Basic web recon methodology
- Testing for common web vulns (XSS, SQLi, IDOR)
- Using enumeration tools effectively
- HTTP request/response analysis
- How dating apps can have security flaws too lol

---

## Tips for Others

- Don't skip the enumeration phase - gobuster found some key directories
- Check robots.txt and source code first
- Test all input fields systematically
- Burp Suite is your friend - intercept everything
- Try simple attacks before getting fancy
- Read error messages carefully - they leak info

---

## Reflection

Honestly pretty easy once you know what to look for. Took me about 20 minutes from start to finish. Good for beginners who are just getting into web app testing.

The theme was fun and the challenge teaches the fundamentals without being too frustrating. Would recommend this as a first CTF challenge for anyone new to web security.

---

**Completed:** March 27, 2026  
**Time taken:** ~20 minutes  
**Difficulty:** ⭐☆☆☆☆
