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
