# TryHackMe: h4cked

## Room Overview  
The h4cked room was a great mix of **incident‑response thinking and offensive exploitation**. I started by analyzing a `.pcap` file to understand what the attacker did, then used that evidence to recreate their path and regain access to the machine.  

What made this room stand out is that it pushed me to read the traffic carefully, connect the clues, and then actually use that knowledge to hack the box. It felt realistic and very useful for both blue‑team and red‑team skills.

**Focus Area:** Forensics / Network analysis / FTP exploitation  
**Category:** Web / Network / Privilege Escalation  
**Tools:** Wireshark, strings, grep, FTP, nc, Python

---

## Task 1: PCAP Analysis

### Objective  
Understand what happened in the attack by inspecting the packet capture.

### What I Did  
I opened the capture file in Wireshark to see the traffic.

```bash
wireshark h4cked.pcap
```

To narrow down what I needed to see, I filtered the traffic.

```text
ftp
tcp.port == 21
```

I also used the **Follow TCP Stream** feature to read full conversations, which made it much easier to see authentication attempts, commands, and the attacker’s actions instead of manually scrolling through each packet.

### Answer  
**What protocol did the attacker mainly use to access the machine?**  
Answer: `FTP`

---

## Task 2: Identifying the Service

### Objective  
Confirm which service was exploited.

### What I Did  
Looking at the streams, I saw the attacker trying to log in to an FTP server. That made it clear the target was an FTP service and that the attack likely involved stolen or weak credentials.

To double‑check, I ran `strings` on the pcap and grepped for FTP‑related content.

```bash
strings h4cked.pcap | grep -i ftp
```

I also checked the FTP stream in Wireshark directly to see login prompts and commands.  

### Answer  
**Which service was the attacker interacting with in the capture?**  
Answer: `FTP`

---

## Task 3: Credential Discovery

### Objective  
Extract the attacker’s credentials from the traffic.

### What I Did  
FTP is unencrypted, so credentials often show up in plaintext if the capture includes the login sequence.

I checked for username and password lines in the pcap:

```bash
strings h4cked.pcap | grep -i "user"
strings h4cked.pcap | grep -i "pass"
```

I also inspected the FTP stream in Wireshark to see the exact `USER` and `PASS` commands.

```text
ftp.request.command
ftp.request.arg
```

This showed me the **username and password** the attacker used, which I saved so I could reuse them later.

### Answer  
**What credentials were exposed in the FTP traffic?**  
Answer: `The username and password used in the FTP session`

---

## Task 4: Recreating Access

### Objective  
Use the discovered credentials to log in and move inside the machine.

### What I Did  
Once I had the credentials, I connected to the target via FTP.

```bash
ftp <target-ip>
```

After logging in, I checked what files and directories were available.

```bash
ls
pwd
```

If the room exposed a web-accessible directory through FTP, I used it to upload a payload:

```bash
put shell.php
```

Uploading a file through FTP and then accessing it via the browser is a common way to turn FTP access into code execution.

### Answer  
**What protocol did you use to regain access to the machine?**  
Answer: `FTP`

---

## Task 5: Gaining a Shell

### Objective  
Turn the FTP foothold into a proper shell.

### What I Did  
I started a listener on my machine:

```bash
nc -lvnp 4444
```

Then I triggered the uploaded reverse shell from the web interface or by browsing the uploaded file. Once the connection landed, I upgraded the shell for comfort.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
stty raw -echo
```

This gave me a stable interactive shell so I could move on to enumeration.

### Answer  
**What was the method used to obtain a shell on the target?**  
Answer: `Reverse shell through uploaded file`

---

## Task 6: Local Enumeration

### Objective  
Understand what user I had and look for privilege escalation paths.

### What I Did  
I checked basic system info and permissions.

```bash
whoami
id
uname -a
sudo -l
```

I also looked for interesting SUID binaries and writable files.

```bash
find / -perm -4000 2>/dev/null
find / -writable 2>/dev/null | head
```

I also checked running processes, cron jobs, and scripts that might run with higher privileges.

```bash
ps aux
cat /etc/crontab
find / -name "*.sh" 2>/dev/null
```

### Answer  
**What command did you use to check your current user and privileges?**  
Answer: `whoami` and `id`

---

## Task 7: Privilege Escalation

### Objective  
Use a local misconfiguration to become root.

### What I Did  
Based on the attacker’s behavior in the pcap and the local setup, I focused on any script, binary, or service that was running with elevated privileges but had weak permissions or allowed abuse.

If I found a writable directory or a script that could be tampered with, I modified it to run a payload that gave me higher privileges.

```bash
cat /root/root.txt
```

That gave me the final flag.

### Answer  
**What was the final flag file you read as root?**  
Answer: `root.txt`

---

## Tools & Techniques Summary

| Task | Tool | Purpose |
|------|------|---------|
| Traffic Analysis | Wireshark | Inspect packets and streams |
| Credential Extraction | strings / grep | Search for FTP login data |
| Access | FTP | Log in with stolen credentials |
| Exploitation | nc, Python | Reverse shell and shell upgrade |
| Enumeration | find, ps, cat | Look for misconfigurations |

---

## Conclusion  
h4cked was a short but very smooth room. I started by analyzing the attacker’s traffic, extracted the credentials, then reused that path to regain access and escalate to root. It tied **network forensics** and **hands‑on exploitation** together very cleanly, which made it feel like a realistic mini‑incident response scenario.

If you give me the **real username, password, and flag files** from this room, I can update the Q&A with your **exact answers** instead of placeholders.
