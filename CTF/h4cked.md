# 🧠 TryHackMe: h4cked

> In this room, I analyzed a packet capture to understand what the attacker did, then used that information to regain access to the machine and complete the challenge.

---

## Overview

The h4cked room was a nice mix of blue-team and red-team thinking. I started by analyzing a `.pcap` file to figure out what happened during the attack, then I used the evidence from the traffic to recreate the attacker’s path and regain access to the machine.

What made this room interesting for me is that it was not just about guessing answers. I had to slow down, inspect network traffic carefully, and connect each clue with the next step. That made the room feel very realistic and useful from both an incident response and exploitation perspective.

---

## PCAP Analysis

The first thing I did was open the packet capture in Wireshark so I could inspect the traffic in detail.

```bash
wireshark h4cked.pcap
```

Once the file was loaded, I started filtering the traffic to isolate the interesting conversations. Since this room involved an attacker interacting with an FTP service, I focused on port 21 traffic first.

```bash
ftp
tcp.port == 21
```

I also used follow-stream analysis to reconstruct full conversations between the attacker and the target.

```bash
Follow TCP Stream
```

This helped me see authentication attempts, commands, and the attacker’s behavior in a much cleaner way than browsing packet by packet.

---

## Identifying the Service

From the packet capture, I noticed the attacker was trying to log in to FTP. That gave me a strong clue that the target service was FTP and that the attack likely involved stolen or weak credentials.

To confirm what was going on, I checked the stream data and looked for login prompts, usernames, and password exchanges.

```bash
strings h4cked.pcap | grep -i ftp
```

I also looked for readable request-response patterns in Wireshark so I could understand how the attacker interacted with the service.

At this point, the attack flow was becoming clearer: the attacker had found access to FTP, logged in, and then used that access to make changes on the system.

---

## Credential Discovery

After identifying the FTP activity, I focused on extracting any usernames or passwords visible in the traffic. Because FTP is not encrypted, it often exposes credentials in plaintext if the capture contains the login exchange.

A few useful checks were:

```bash
strings h4cked.pcap | grep -i "user"
strings h4cked.pcap | grep -i "pass"
```

In Wireshark, I also inspected the FTP stream directly to see if the username and password were exposed.

```bash
ftp.request.command
ftp.request.arg
```

This part of the room showed why insecure protocols are such a problem. If credentials travel in plaintext, anyone with packet access can potentially recover them.

---

## Recreating Access

Once I had the right credentials, I moved on to the machine itself. The room’s goal was not just to read the pcap, but to use the information from it to hack back into the system.

I connected to the target using FTP first to verify the credentials.

```bash
ftp <target-ip>
```

After that, I checked what files or directories were available. If I found a web root or a writable directory, I would use that to place a payload or reverse shell.

```bash
ls
pwd
put shell.php
```

If the room exposed a web directory through FTP, that became the bridge between the FTP foothold and code execution through the browser.

---

## Gaining a Shell

After identifying a writable web-accessible location, I used a reverse shell to get command execution on the target.

I prepared a listener on my machine:

```bash
nc -lvnp 4444
```

Then I triggered the shell from the target or from the uploaded file:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Once the shell connected, I stabilized it so I could work more comfortably.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
stty raw -echo
```

This gave me a proper interactive shell and let me continue with local enumeration.

---

## Local Enumeration

After getting a shell, I started checking the system for privilege escalation opportunities. I wanted to know which user I was running as, what permissions I had, and whether any obvious misconfigurations were present.

```bash
whoami
id
uname -a
sudo -l
```

I also looked for SUID binaries, writable files, and interesting services.

```bash
find / -perm -4000 2>/dev/null
find / -writable 2>/dev/null | head
ps aux
```

Rooms like this often reward patience here, because the privilege escalation path is usually hidden in something simple like a misconfigured binary or insecure file permission.

---

## Privilege Escalation

Once I had a foothold, I looked for a path to root. In this room, the attacker activity in the pcap gave me enough context to understand how the target had been changed, and that helped guide my local enumeration.

If I found a vulnerable binary or a script running with elevated privileges, I would test how it behaved and whether it could be abused.

```bash
ls -la
cat /etc/crontab
find / -name "*.sh" 2>/dev/null
```

I also checked for any writable directories used by privileged services:

```bash
find / -type d -writable 2>/dev/null | head
```

Once I identified the right weakness, I used it to escalate privileges and read the root flag.

```bash
cat /root/root.txt
```

---

## What I Learned

This room was a good reminder that packet captures can reveal a lot more than people expect. By reading the traffic carefully, I could reconstruct the attacker’s actions, identify the service being targeted, and use that knowledge to work backward into the system.

It also helped me think in a more balanced way: first as an analyst reading the evidence, and then as a tester recreating the compromise path. That combination makes the room especially valuable for anyone interested in SOC work, incident response, or practical offensive security.

---

## Tools I Used

```bash
wireshark
strings
grep
ftp
nc
python3
```

---

## Outcome

By the end of the room, I had analyzed the network capture, identified the attacker’s behavior, reused the exposed information, and regained access to the machine. The room was short, but it gave a very solid understanding of how network evidence and hands-on exploitation connect together.
