# TryHackMe: Investigating Windows — Writeup
 
## Room Overview
 
Investigating Windows is a forensics room focused on analyzing a Windows machine compromise.
I had to examine logs, event files, scheduled tasks, network activity, and other evidence to
reconstruct the attacker's path and answer detailed questions about the incident.
 
| Field | Info |
|-------|------|
| Target OS | Windows Server 2016 |
| Focus Area | Windows Forensics / Incident Response |
| Tools Used | Wireshark, Event Viewer, strings, grep, FTP, netstat, scheduled task analysis |
 
---
 
## Task 1: Initial Analysis
 
### Objective
Identify the Windows version and basic system information.
 
### What I Did
I started by checking the system information from the provided evidence (logs, screenshots, etc.).
 
### Answer
**What version of Windows machine is this?**
> Windows Server 2016
 
---
 
## Task 2: User Activity
 
### Objective
Identify who logged into the system and when.
 
### What I Did
I looked at authentication logs and login events to see which users accessed the system.
 
### Answers
**Which user logged in last?**
> Administrator
 
**When did John log into the system?**
> 02/03/2021 13:55 AM
 
---
 
## Task 3: Network Activity
 
### Objective
Analyze network connections and identify suspicious activity.
 
### What I Did
I used Wireshark and network logs to see what connections were made.
 
### Answers
**What IP address did the first connection attempt to?**
> 10.10.34.3
 
**What port did the attacker open?**
> 1348
 
---
 
## Task 4: Administrative Users
 
### Objective
Identify users with administrative privileges.
 
### What I Did
I checked the local users and groups to see who had admin rights.
 
### Answer
**What two users had administrative privileges (other than Administrator)? List in alphabetical order.**
> Jerry, Jenny
 
---
 
## Task 5: Scheduled Tasks
 
### Objective
Find malicious scheduled tasks and understand their behavior.
 
### What I Did
I analyzed the scheduled tasks to find anything suspicious.
 
### Answers
**What is the name of the scheduled task that is malicious?**
> UpdateTask
 
**What is the task trying to run daily?**
> cmd.exe
 
**What port did this file listen on locally for?**
> 1348
 
---
 
## Task 6: Compromise Timeline
 
### Objective
Build a timeline of the attack.
 
### What I Did
I correlated events from logs to understand the sequence.
 
### Answers
**When did Jerry log on?**
> 08/02/2021
 
**At what date did the compromise take place?**
> 08/02/2021
 
**During what time did Windows first gain special privileges to a new logon?**
> 08/02/2019 14:48 PM
 
---
 
## Task 7: Attack Tools
 
### Objective
Identify the tools and techniques used by the attacker.
 
### What I Did
I looked at process execution logs and network activity.
 
### Answers
**What tool was used to get Windows passwords?**
> Mimikatz
 
**What was the IP address of the external control and command server?**
> 185.243.218.42
 
**What is the full name of the shell uploaded via the web server?**
> nc64.exe
 
---
 
## Task 8: DNS Poisoning
 
### Objective
Identify DNS manipulation activity.
 
### What I Did
I checked DNS queries and resolution activity.
 
### Answer
**What site was targeted in the DNS poisoning?**
> google.com
 
---
 
## Attack Timeline Summary
 
1. 08/02/2021 — Jerry logs on
2. Compromise occurs — Mimikatz extracts credentials
3. Attacker uploads `nc64.exe` via web server
4. Creates malicious scheduled task `UpdateTask`
5. Opens reverse shell on port `1348`
6. DNS poisoning targets `google.com`
7. Gains SYSTEM privileges via new logon
 
---
 
## Tools & Techniques Summary
 
| Task | Tool / Technique | Evidence Location |
|------|-----------------|-------------------|
| System Info | Event Logs | Windows Server 2016 |
| User Enumeration | Local Groups | Administrator, Jerry, Jenny |
| Scheduled Tasks | Task Scheduler | UpdateTask → cmd.exe |
| Network Activity | Wireshark / Logs | 185.243.218.42, port 1348 |
| Credential Theft | Process Logs | Mimikatz |
| Persistence | Scheduled Tasks | Daily cmd.exe execution |
| C2 Communication | Network Logs | nc64.exe reverse shell |
| DNS Manipulation | DNS Logs | google.com poisoning |
 
---
 
## Key Commands & Analysis
 
```bash
# PCAP Analysis
wireshark h4cked.pcap
tcp.port == 21
 
# String extraction
strings h4cked.pcap | grep -i "user"
strings h4cked.pcap | grep -i "pass"
 
# FTP access
ftp <target-ip>
 
# Network listener
nc -lvnp 1348
 
# Shell upgrade
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
 
---
 
## Conclusion
 
Investigating Windows was a solid forensics room that forced me to connect multiple pieces of
evidence — logs, network traffic, scheduled tasks, and process activity — to build the full
attack story. It was great practice for understanding Windows incident response and recognizing
common persistence and lateral movement techniques.
 
The room showed exactly how attackers chain simple footholds (FTP credentials) into full
compromise through credential dumping, webshells, scheduled tasks, and DNS manipulation. A
very realistic attack chain overall.
