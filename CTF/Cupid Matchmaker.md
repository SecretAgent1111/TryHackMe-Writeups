# Cupid’s Matchmaker – TryHackMe Writeup

## 1. Introduction

Cupid’s Matchmaker is a beginner web challenge from TryHackMe’s “Love At First Breach 2026” event. The goal is to exploit a stored XSS vulnerability in a matchmaking site that promises “no AI, just real humans reading your personality survey.” This hint suggests that someone privileged will later view the data I submit, which makes the survey form a strong candidate for stored XSS exploitation.

By the end of the room, the admin’s session cookie is obtained as the flag, using client‑side JavaScript payload injected into the survey and triggered in the reviewer’s browser.

---

### Suggested screenshot here:
- Homepage of Cupid’s Matchmaker, showing the description and “Take survey” button.
- Example filename: `screenshots/cupid-homepage.png`

---

## 2. Recon and Initial Analysis

After starting the room, I accessed the web application from the provided URL and explored the main pages. I first browsed the homepage to understand the flow: the user fills out a personality survey, then a “human” reviews the submission, and finally the user is matched based on the profile.

I then clicked “Take survey” and inspected the form fields, focusing on any free‑text areas where I could inject arbitrary content. The structure of the form and the mention of human review strongly suggested that user input was stored and later rendered somewhere in the application, which is the classic setup for a stored XSS attack.

---

### Suggested screenshot here:
- The survey form page, highlighting input fields (especially the long‑text area like “About you” or “Describe yourself”).
- Example filename: `screenshots/cupid-survey-form.png`

---

## 3. Testing for Stored XSS

To test for stored XSS, I submitted a simple proof‑of‑concept payload in one of the free‑text fields while keeping the rest of the form filled normally. I then submitted the survey and received a confirmation message saying the survey would be reviewed shortly.

From my own browser, I did not see any immediate effect such as an alert popup, which is expected behavior for stored XSS. The payload is stored on the server and only executed later when the reviewer (admin/bot) opens the submission in their browser. This behavior confirmed that the vulnerability type was stored XSS rather than reflected or DOM‑based XSS.

---

### Suggested screenshot here:
- The survey after submission showing the confirmation message.
- Optionally, a browser devtools console where you can show the lack of alert on your side.
- Example filename: `screenshots/survey-submitted-confirm.png`

---

## 4. Setting Up the Listener and Final Payload

Next, I prepared a listener on my machine to capture the admin’s cookie once the payload executed. I chose a listener port and started a simple HTTP/netcat listener that would catch any incoming requests. The idea was to have the admin’s browser send a request containing `document.cookie` as a URL parameter to my IP.

I then crafted a JavaScript payload that would silently send the admin’s cookie to my listener by creating an image request to my server. I placed this payload into the same free‑text form field and submitted the survey again, waiting for the reviewer to open my submission.

---

### Suggested screenshot here:
- Terminal with your listener running (e.g., `nc -lvnp 8080` or Python HTTP server).
- Example filename: `screenshots/nc-listener.png`

---

## 5. Capturing the Flag and Lessons Learned

After the reviewer opened the submission, my listener received an HTTP GET request containing the admin’s session cookie as a query parameter. From that request, I copied the exact value that served as the flag and pasted it into the answer box on TryHackMe to complete the room.

From this challenge, I learned how dangerous stored XSS can be when user‑generated content is viewed by admins or privileged users. The scenario also reinforced the importance of using `HttpOnly` cookies and proper input sanitization/output encoding to prevent cookie theft through XSS. Practicing this flow helped me better understand how to spot similar stored XSS opportunities in real‑world applications and how to design payloads that quietly exfiltrate sensitive data.

---

### Suggested screenshot here:
- The captured HTTP request in your listener showing the cookie with the flag.
- Example filename: `screenshots/flag-extracted.png`
