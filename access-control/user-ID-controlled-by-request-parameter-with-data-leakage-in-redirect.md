● Lab Report: User ID Controlled by Request Parameter with Data Leakage in Redirect
                                                                                          
  Platform: PortSwigger Web Security Academy                                                                               
  Category: Broken Access Control — IDOR with Data Leakage in Redirect                                                     
  Difficulty: Low                                                                                                          
  Date Solved: 01 May 2026                                                                                                 
  Lab URL: https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter-with-data-leakag
  e-in-redirect

  ---
  Vulnerability Summary

  The application attempts to block unauthorized account access by redirecting users away when the id parameter does not
  match their session. On the surface this looks like a working protection — the user gets sent back to the home page.
  However, the server renders the full account page including the target user's API key in the body of the redirect
  response before issuing the 302. Burp Suite captures this response body before following the redirect, exposing all
  sensitive data that was supposed to be protected.

  The redirect is a cosmetic fix. The server already processed and returned the sensitive data — it just told the browser
  to go elsewhere immediately after.

  OWASP Category: A01:2021 — Broken Access Control
  CWE: CWE-601 (URL Redirection) / CWE-200 (Exposure of Sensitive Information) / CWE-284 (Improper Access Control)

  ---
  How Serious Is It?

  Very serious — and deceptive. This vulnerability is dangerous precisely because it looks protected. A developer testing
  in a browser sees the redirect and assumes the unauthorized access was blocked. They never see the response body. An
  attacker using Burp Suite sees everything the browser discards. Any sensitive data included in a redirect response body
  is fully exposed to anyone intercepting traffic.

  ---
  Discovery

  Reconnaissance steps taken:

  1. Logged in with provided credentials — observed URL after login: /my-account?id=wiener
  2. Intercepted the login request in Burp Suite — changed username in the login parameters — received 302 unauthorized, no
   data exposed at that stage
  3. Logged in successfully, reloaded the account page with Burp Suite intercepting
  4. Changed id=wiener to id=carlos in Burp Repeater
  5. Server returned a 302 redirect to home page — but the response body contained carlos's full account page including his
   API key
  6. Copied the API key from the redirect response body — submitted it — lab solved

  What indicated something was wrong:

  The 302 redirect response had a non-empty body. A proper redirect that is genuinely blocking access should return an
  empty body or a generic message. Seeing account data in the body of a redirect response immediately signals that the
  server processed the request fully before deciding to redirect — the protection is applied too late.

  ---
  Exploitation

  Step 1 — Identify the vulnerable parameter

  Logged in as wiener. Observed account URL: /my-account?id=wiener. Sent request to Burp Repeater.

  Step 2 — Modify the ID parameter

  Changed the request in Repeater:

  GET /my-account?id=carlos HTTP/2
  Host: [lab-host].web-security-academy.net
  Cookie: session=[wiener-session-cookie]

  Step 3 — Inspect the redirect response

  Server responded with 302 Found — redirect to home page. However, the response body contained carlos's full account page:

  HTTP/2 302 Found
  Location: /login
  Content-Type: text/html; charset=utf-8

  ...
  <div class="account-info">
      Your API Key is: [carlos-api-key]
  </div>
  ...

  Step 4 — Extract and submit

  Copied carlos's API key directly from the 302 response body. Submitted it — lab solved.

  Key technical detail: A browser automatically follows the 302 and discards the response body — the user never sees it.
  Burp Repeater does not follow redirects by default, so the full response body is visible. This is why the vulnerability
  is invisible to casual testing but trivially exploitable with an intercepting proxy.

  ---
  Impact Analysis

  What an attacker can do:
  - Access any user's sensitive account data by changing the id parameter — the redirect does not prevent this
  - Extract API keys, personal information, or any data the account page renders
  - Automate the attack to harvest data from every user account by cycling through usernames or IDs
  - The attack is silent — server logs record only a 302 redirect, which appears like a blocked access attempt

  Who is affected:
  - Users: Every account whose data is rendered server-side before the redirect is fully exposed
  - Business: What appears in logs as a working security control is actually leaking data on every attempt
  - Security team: A log review showing 302 responses may give false confidence that unauthorized access attempts were
  successfully blocked

  Worst-case real-world scenario:
  An attacker writes a script to cycle through all usernames, collecting the 302 response body for each one. Every response
   contains that user's private data. The server logs show hundreds of blocked 302 redirects — the security team sees the
  logs and concludes the system is working correctly. Meanwhile the attacker has exfiltrated data for every user on the
  platform.

  ---
  Root Cause

  The developer added a redirect as a security control but implemented it in the wrong order:

  ❌ Wrong order (vulnerable):
  1. Render full account page with user data
  2. Check if session matches the id parameter
  3. If not — redirect to home page
     (data already rendered in the response body)

  ✅ Correct order:
  1. Check if session matches the id parameter
  2. If not — redirect immediately with empty body
  3. Only render account data if check passes

  The access control check ran after the data was already included in the response. By the time the redirect was issued,
  the damage was done.

  ---
  Recommended Fix

  1. Check authorization before rendering any data — the session ownership check must be the first operation on any account
   endpoint, before any user data is fetched or included in the response
  2. Return an empty body on redirects — a 302 response used as an access control mechanism must have no body, or a generic
   message only. Never include the protected resource in the redirect response
  3. Use server-side session to derive account context — the account page should render data for the user tied to the
  active session, not for whatever id value the client sends
  4. Test with an intercepting proxy, not just a browser — security testing that only uses a browser will miss data leakage
   in redirect bodies. Always inspect raw responses

  OWASP Reference: https://owasp.org/Top10/A01_2021-Broken_Access_Control/
  CWE Reference: https://cwe.mitre.org/data/definitions/200.html

  ---
  Key Takeaway

  ▎ A redirect is not access control — it is a navigation instruction to the browser. If the server renders sensitive data
  ▎ before issuing the redirect, that data is already in the response body and fully visible to anyone using Burp Suite.
  ▎ Always check authorization first, render data second. And never trust that a 302 in the logs means an attack was
  ▎ blocked — check what was in the response body.
