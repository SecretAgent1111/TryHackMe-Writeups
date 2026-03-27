# TryHackMe - Valentine Writeup

## Room Information
**Room Name:** Valentine  
**Difficulty:** Entry-Level / Easy  
**Category:** Web Application Security  
**Platform:** TryHackMe  
**Event:** Love at First Breach 2026

---

## Description
Valentine is the entry-level web challenge for the Love at First Breach 2026 CTF event. This beginner-friendly room introduces fundamental web application vulnerabilities through a dating/matchmaking themed web application. It serves as the perfect starting point for those new to web exploitation and CTF challenges.

---

## Learning Objectives
- Web application reconnaissance
- Identifying common web vulnerabilities
- Basic exploitation techniques
- Understanding HTTP requests and responses
- Introduction to web application security testing

---

## Initial Access & Setup

### Step 1: Deploy the Machine
1. Click "Start Machine" on TryHackMe
2. Wait for the machine to fully deploy (typically 1-2 minutes)
3. Note the IP address provided: `<target-ip>`

### Step 2: Verify Connectivity
```bash
ping <target-ip>
```

---

## Reconnaissance

### Web Application Discovery

Navigate to the target in your browser:
```
http://<target-ip>
```

### Initial Observations
- Identify the application type (dating/matchmaking site)
- Explore visible pages and functionality
- Check for:
  - Login/Registration forms
  - User profiles
  - Search functionality
  - Upload features
  - Contact/messaging systems

### Directory Enumeration

Use GoBuster to discover hidden directories and files:

```bash
gobuster dir -u http://<target-ip> -w /usr/share/wordlists/dirb/common.txt -x php,txt,html,js
```

**Alternative tools:**
```bash
# Using ffuf
ffuf -u http://<target-ip>/FUZZ -w /usr/share/wordlists/dirb/common.txt

# Using dirb
dirb http://<target-ip> /usr/share/wordlists/dirb/common.txt
```

### Port Scanning

Perform a quick Nmap scan to identify open ports:

```bash
nmap -sV -sC <target-ip>
```

**Common findings:**
- Port 80 (HTTP)
- Port 443 (HTTPS)
- Additional services may be present

---

## Vulnerability Analysis

### Testing for Common Web Vulnerabilities

#### 1. Input Validation Testing

Test all input fields for:
- SQL Injection
- Cross-Site Scripting (XSS)
- Command Injection
- Path Traversal

**XSS Test Payloads:**
```html
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
```

**SQL Injection Test Payloads:**
```sql
' OR '1'='1
' OR 1=1--
admin'--
' UNION SELECT NULL--
```

#### 2. Authentication Testing

**Check for:**
- Default credentials
- Weak password policies
- Session management issues
- Password reset vulnerabilities

**Common default credentials to try:**
```
admin:admin
admin:password
root:root
test:test
```

#### 3. Access Control Testing

**Test for Insecure Direct Object References (IDOR):**
```
GET /profile?id=1
GET /profile?id=2
GET /user/1
GET /user/2
```

**Modify URL parameters:**
```
/view?file=user1.txt
/view?file=user2.txt
/view?file=../admin.txt
```

---

## Exploitation

### Finding the Vulnerability

#### Step 1: Analyze Application Behavior

Use Burp Suite or browser DevTools to:
- Intercept HTTP requests
- Analyze responses
- Identify interesting endpoints
- Check for hidden parameters

#### Step 2: Test Discovered Endpoints

For each endpoint found during enumeration:
1. Test with various payloads
2. Observe error messages
3. Check for information disclosure
4. Look for unexpected behavior

#### Step 3: Exploit the Weakness

**Example scenarios:**

**Scenario A: Direct File Access**
```bash
curl http://<target-ip>/admin.php
curl http://<target-ip>/config.php
curl http://<target-ip>/backup.sql
```

**Scenario B: Parameter Manipulation**
```bash
curl "http://<target-ip>/view.php?file=flag.txt"
curl "http://<target-ip>/profile.php?user=admin"
```

**Scenario C: Cookie Manipulation**
```bash
# Modify cookie values
curl -b "role=admin" http://<target-ip>/dashboard
```

---

## Capturing the Flag

### Flag Location

Depending on the challenge, flags may be located in:
- Hidden files
- Database entries
- Admin panels
- Source code comments
- Environment variables

### Common Flag Formats
```
THM{...}
flag{...}
FLAG{...}
```

### Extraction Techniques

1. **Direct Access:**
   ```bash
   curl http://<target-ip>/flag.txt
   ```

2. **Source Code Review:**
   ```bash
   # View page source
   curl http://<target-ip> | grep -i "flag"
   ```

3. **Using Exploitation:**
   ```bash
   # After gaining access
   cat /var/www/html/flag.txt
   ```

---

## Tools Used

| Tool | Purpose | Command Example |
|------|---------|------------------|
| GoBuster | Directory enumeration | `gobuster dir -u <url> -w <wordlist>` |
| Nmap | Port scanning | `nmap -sV <ip>` |
| cURL | HTTP requests | `curl http://<target-ip>` |
| Burp Suite | Traffic interception | GUI-based |
| Browser DevTools | Frontend analysis | F12 in browser |
| Wfuzz | Web fuzzing | `wfuzz -u <url> -w <wordlist>` |

---

## Methodology Summary

```
1. Reconnaissance
   ├── Access web application
   ├── Browse visible pages
   └── Enumerate directories

2. Vulnerability Discovery
   ├── Test input fields
   ├── Check authentication
   ├── Analyze access controls
   └── Identify weaknesses

3. Exploitation
   ├── Craft payload
   ├── Execute attack
   └── Verify success

4. Flag Capture
   ├── Locate flag
   ├── Extract data
   └── Submit answer
```

---

## Common Pitfalls & Tips

### Mistakes to Avoid
- Skipping reconnaissance phase
- Not testing all input fields
- Ignoring error messages
- Overlooking obvious directories (robots.txt, sitemap.xml)
- Not checking page source code

### Success Tips
- **Be systematic:** Test everything methodically
- **Take notes:** Document your findings
- **Read error messages:** They often reveal valuable information
- **Check the basics first:** robots.txt, .git, backups
- **Try simple payloads before complex ones**
- **Use Burp Suite:** Intercept and modify requests

---

## Skills Developed

By completing this room, you'll gain experience in:

- ✓ Web application reconnaissance
- ✓ Directory and file enumeration
- ✓ HTTP request/response analysis
- ✓ Basic vulnerability identification
- ✓ Entry-level exploitation techniques
- ✓ Tool familiarity (GoBuster, Burp Suite, cURL)
- ✓ CTF methodology and approach

---

## Next Steps

After completing Valentine, try these rooms:

**Similar Difficulty:**
- Hidden Deep Into My Heart
- Corp Website
- TryHartMe beginner rooms

**Slightly More Advanced:**
- Signed Messages
- CupidBot
- Speed Chatting

---

## References & Resources

**Web Security Fundamentals:**
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
- [HackTricks](https://book.hacktricks.xyz/)

**Tools Documentation:**
- [GoBuster GitHub](https://github.com/OJ/gobuster)
- [Burp Suite Docs](https://portswigger.net/burp/documentation)
- [Nmap Reference](https://nmap.org/book/man.html)

**CTF Resources:**
- [TryHackMe Learning Paths](https://tryhackme.com/paths)
- [PicoCTF Resources](https://picoctf.org/resources)

---

## Conclusion

Valentine serves as an excellent introduction to web application security and CTF challenges. The room teaches fundamental concepts that form the foundation for more advanced exploitation techniques. By completing this challenge, you've taken your first step into the world of offensive security and web application penetration testing.

Remember: These skills should only be used ethically and on systems you have permission to test.

---

**Room Completed:** ✓  
**Difficulty Rating:** ⭐☆☆☆☆ (Entry-Level)  
**Recommended For:** Absolute beginners, CTF newcomers  
**Time to Complete:** 15-30 minutes

---

*Writeup created for educational purposes as part of TryHackMe's Love at First Breach 2026 event.*
