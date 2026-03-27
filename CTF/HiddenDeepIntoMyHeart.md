# Hidden Deep Into My Heart - TryHackMe Writeup

Just wrapped up the "Hidden Deep Into My Heart" room from Love at First Breach 2026. This one was all about directory bruteforcing and finding hidden paths - basically hide and seek with web directories lol.

## Room Details
- **Name:** Hidden Deep Into My Heart
- **Difficulty:** Entry-Level
- **Category:** Web Security / Directory Enumeration
- **Platform:** TryHackMe

---

## Initial Setup

Started the machine like usual. Got my target IP and did a quick ping check to make sure it's reachable.

```bash
ping <target-ip>
```

All good, machine is up.

---

## First Look

Opened the IP in my browser to see what we're working with. Pretty basic landing page - nothing too fancy. Looked around the visible pages but didn't find much interesting stuff on the surface.

Checked the page source and robots.txt out of habit but nothing jumped out immediately.

---

## Directory Bruteforcing

This is where the real challenge starts. The room name basically tells you what to do - find what's hidden deep in the directories.

Fired up gobuster with a common wordlist:

```bash
gobuster dir -u http://<target-ip> -w /usr/share/wordlists/dirb/common.txt
```

Got some results but nothing too crazy yet. Decided to dig deeper with a bigger wordlist:

```bash
gobuster dir -u http://<target-ip> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```

This took a bit longer but started finding some interesting directories.

---

## Going Deeper

Found a few subdirectories that weren't linked anywhere on the main site. Started exploring them one by one.

Tried accessing them directly:
```bash
curl http://<target-ip>/hidden-directory
```

Some directories had more directories inside them. Had to run gobuster again on those subdirectories:

```bash
gobuster dir -u http://<target-ip>/found-directory -w /usr/share/wordlists/dirb/common.txt
```

It's like a maze of directories - the "deep" part of the title makes sense now.

---

## Finding the Hidden Path

After going through multiple levels of directories, finally found a path that wasn't obvious at all. Had to be pretty persistent with the enumeration.

The key was:
1. Not giving up after the first gobuster scan
2. Checking inside discovered directories for more subdirectories  
3. Using different wordlists when one didn't work
4. Being patient - this stuff takes time

---

## Getting the Flag

Once I found the deeply nested directory, navigated to it in the browser. The flag was sitting there in a file.

```bash
curl http://<target-ip>/very/deep/hidden/path/flag.txt
```

Flag format: `THM{...}`

Submitted it and got the points.

---

## Tools I Used

- **GoBuster** - Main tool for directory enumeration
- **cURL** - Quick way to check directories
- **Firefox** - For browsing and checking found paths
- **DirBuster wordlists** - Different wordlists for different scan depths

---

## What I Learned

This room really hammered home some important lessons:
- Directory bruteforcing is essential for web pentesting
- Sometimes you need to go multiple levels deep
- Different wordlists give different results
- Patience is key - don't stop at the first scan
- Hidden paths won't always show up in robots.txt or source code

---

## Tips for Others

- Start with common.txt but don't stop there
- If you find a directory, scan inside it too
- Use the `-x` flag with gobuster to check for specific file extensions
- Try medium and large wordlists if small ones don't work
- Keep notes of what directories you've already checked
- The room name is a hint - go DEEP into the directory structure

---

## Reflection

Pretty fun challenge that teaches a really practical skill. In real pentests, you'll definitely need to enumerate directories to find admin panels, config files, backups, etc. that aren't meant to be public.

Took me maybe 30 minutes including the time waiting for scans to complete. Not super difficult but good practice for building the enumeration mindset.

---

**Completed:** March 27, 2026  
**Time taken:** ~30 minutes  
**Difficulty:** ⭐⭐☆☆☆
