# 🧠 TryHackMe: CMSpit

> **Room Focus:** Web application exploitation, CMS enumeration, user enumeration, password reset abuse, and privilege escalation.  
> **Room Link:** https://tryhackme.com/room/cmspit  

---

## Overview

CMSpit is a TryHackMe room centered around attacking a vulnerable content management system and then using the foothold to move toward privilege escalation. The room is a good example of how small weaknesses in a web application can chain together into full system compromise when security controls are missing or misconfigured.

What I liked about this room is that it feels practical. It does not rely on one single trick; instead, it encourages proper enumeration, careful observation of the application, and a step-by-step mindset that mirrors real penetration testing work.

---

## Initial Enumeration

I began with standard reconnaissance to understand what the target was exposing. A basic Nmap scan helped confirm the open services and pointed me toward the web application as the main entry point.

```bash
nmap -sC -sV <target-ip>
```

From the scan results, the interesting service was the web server on port 80, so I shifted my focus there. At this stage, the goal was not to guess the exploit immediately, but to understand the application structure, pages, and visible functionality.

---

## CMS Identification

When I opened the website, the first thing I noticed was that it was built on a content management system. The login page and page source gave away useful clues about the platform, which is often the first real lead in CMS-based challenges.

By checking the HTML source, referenced assets, and visible interface elements, I was able to identify the CMS and determine that the room was built around exploiting weaknesses in that system rather than attacking the operating system directly.

This is a good reminder that in real assessments, small details in page source, comments, asset names, and hidden endpoints often reveal much more than the visible page itself.

---

## Version Discovery

After identifying the CMS, I looked for the exact version. Version discovery matters because many CMS vulnerabilities are version-specific, and a correct version match can quickly narrow down the attack surface.

In this room, the version could be inferred from application behavior and frontend references. That kind of version leakage is common in real environments when developers leave asset version tags or build information exposed.

This step helped me connect the application to known vulnerabilities and confirmed that the room was likely expecting a public exploit chain rather than a custom zero-day approach.

---

## User Enumeration

The next part of the room focused on user enumeration. This was one of the most important stages because it showed how account recovery and login-related features can accidentally leak valid usernames.

I tested the application’s authentication and reset-related functionality carefully. Rather than brute forcing blindly, I observed how the application responded to valid and invalid input. Differences in response text, timing, or behavior can reveal whether a username exists.

That is exactly the kind of weakness an attacker would love to find. If a system leaks whether an account exists, it becomes much easier to target specific users with password reset abuse, phishing, or credential attacks.

---

## Password Reset Abuse

After discovering how the platform handled usernames, I moved to the password reset logic. This was a key step in the room because the reset workflow could be abused to gain access without needing the original password.

I followed the reset process and paid attention to how tokens, requests, and validation behaved. In insecure applications, reset mechanisms are often easier to exploit than the main login form because they are designed for convenience and sometimes lack strict verification.

This room demonstrated how a weak password reset flow can become a direct path into an account. In practice, this is one of the clearest examples of why account recovery features must be designed with the same care as login controls.

---

## Foothold Acquisition

Once account access was achieved, the focus shifted from web exploitation to obtaining a foothold on the target host. This is where web app attacks often transition into system-level compromise.

At this point, I treated the environment like a real engagement: I checked what access the compromised account had, what application functions were available, and whether any uploaded data, exposed secrets, or admin-only features could be abused further.

This part of the room reinforced an important lesson: gaining access to a CMS account does not end the attack. It often opens the door to configuration files, internal functions, or service-level privileges that were never meant to be public.

---

## Privilege Escalation

After establishing a foothold, I moved into local enumeration to look for privilege escalation paths. This included checking system information, permissions, scheduled tasks, writable paths, SUID binaries, and any application-specific files that might reveal elevated access routes.

Privilege escalation is where many rooms become more “realistic,” because the attacker must use host-level clues instead of relying only on the web exploit. The room pushed me to think about how misconfigurations, weak file permissions, or insecure service behavior can turn limited access into full control.

That progression from web entry to local escalation is exactly what makes CMSpit valuable as a practice room.

---

## Security Lessons

This room highlighted several important security lessons that matter both in labs and in real environments:

- User enumeration is dangerous because it helps attackers focus their efforts.
- Password reset flows must be designed carefully and tested thoroughly.
- CMS versions should not be easily exposed through frontend assets or metadata.
- Every account should follow least privilege.
- Web access does not have to be full compromise, but it can become full compromise if internal files and permissions are weak.

For me, the biggest takeaway was that good enumeration leads to good exploitation. The better you understand the application, the easier it becomes to find the weak point in the chain.

---

## Tools Used

```bash
nmap
curl
browser dev tools
directory enumeration tools
manual HTTP testing
```

I kept the process simple and methodical because rooms like this reward observation more than speed.

---

## Outcome

CMSpit was a strong exercise in web application security and privilege escalation thinking. It reinforced the value of careful enumeration, understanding CMS behavior, and testing features like login and password reset with a security mindset.

The room also felt useful from a blue-team perspective because it shows exactly how attackers think: identify the CMS, map the attack surface, test user handling, abuse recovery logic, and then pivot into the host.
