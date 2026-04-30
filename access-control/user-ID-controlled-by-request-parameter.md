  Lab Report: User ID Controlled by Request Parameter
                                                                                          
  Platform: PortSwigger Web Security Academy                                                                               
  Category: Broken Access Control — Insecure Direct Object Reference (IDOR)                                                
  Difficulty: Low                                                                                                          
  Date Solved: 30 April 2026                                                                                               
  Lab URL: https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter

  ---
  Vulnerability Summary

  The application uses a predictable username-based parameter in the URL to identify which account page to display. After
  login, the user is directed to /my-account?id=wiener. There is no server-side check verifying that the logged-in user
  matches the id value in the request. By simply changing id=wiener to id=carlos, an attacker can access any other user's
  account page — including their API key — without any additional credentials or permissions.

  This is a classic Insecure Direct Object Reference (IDOR) vulnerability.

  OWASP Category: A01:2021 — Broken Access Control
  CWE: CWE-639 (Authorization Bypass Through User-Controlled Key) / CWE-284 (Improper Access Control)

  ---
  How Serious Is It?

  Very serious in real applications. Most applications expose far more than just an API key on a profile page — billing
  information, personal details, private messages, addresses, and transaction history are all common. An attacker only
  needs one valid account to enumerate every other user's data by cycling through usernames or IDs. No special tools are
  required beyond a browser.

  ---
  Discovery

  Reconnaissance steps taken:

  1. Logged in with provided credentials (wiener)
  2. Observed the post-login URL: /my-account?id=wiener
  3. Noticed the API key was displayed directly on the account page
  4. Intercepted the request in Burp Suite and sent it to Repeater
  5. Changed id=wiener to id=carlos in the request
  6. Server returned carlos's account page including his API key — no authorization error

  What indicated something was wrong:

  The id parameter in the URL was a plain username — predictable and user-controlled. The moment a user-controlled value is
   the only thing determining which account data is returned, and there is no server-side ownership check, IDOR exists. The
   application trusted the parameter completely without verifying it matched the active session.

  ---
  Exploitation

  1. Logged in as wiener with provided credentials
  2. Observed URL after login: /my-account?id=wiener
  3. Intercepted the account page request in Burp Suite, sent to Repeater
  4. Modified the request:
  GET /my-account?id=carlos HTTP/2
  Host: [lab-host].web-security-academy.net
  Cookie: session=[wiener-session-cookie]
  5. Forwarded — server returned carlos's account page with his API key visible in the response
  6. Copied carlos's API key and submitted it — lab solved

  Result: Full access to another user's account page and API key achieved by changing one parameter value.

  ---
  Impact Analysis

  What an attacker can do:
  - Access any user's account page by guessing or enumerating usernames
  - Steal API keys, personal information, or any data displayed on the profile page
  - Cycle through all usernames systematically to harvest data at scale
  - Use stolen API keys to authenticate as other users in connected systems

  Who is affected:
  - Users: Every account on the platform is exposed — not just one
  - Business: Mass data breach risk — all user data accessible to any logged-in attacker
  - Compliance: Depending on the data exposed, this could trigger GDPR, HIPAA, or other regulatory violations

  Worst-case real-world scenario:
  An attacker registers a free account, writes a simple script to cycle through common usernames or sequential IDs, and
  silently harvests account data for every user on the platform — API keys, emails, billing details, personal data. The
  server logs show nothing unusual because every request looks like a legitimate authenticated user viewing a profile page.
   The breach is only discovered when stolen data appears elsewhere.

  ---
  Root Cause

  The server used the client-supplied id parameter to look up and return account data without verifying that the requesting
   session owned that account. The session cookie authenticated the user as wiener — but the server never checked whether
  wiener's session was allowed to view carlos's data.

  Authentication and authorization are separate checks. Being logged in proves who you are. It does not automatically prove
   you are allowed to access every resource you request.

  ---
  Recommended Fix
  Recommended Fix

  1. Derive the user identity from the session, not the URL — the server should use the session cookie to determine which
  account to display, ignoring the id parameter entirely for the account owner view
  2. If the id parameter must exist, validate it server-side — check that the value in id matches the user tied to the
  active session before returning any data
  3. Apply ownership checks on every object reference — any time a request references a resource by ID, the server must
  verify the requesting user owns or has explicit permission to access that resource
  4. Use unpredictable IDs as defense-in-depth — replacing usernames with GUIDs makes enumeration harder, but this is a
  supplement to authorization checks, never a replacement

  OWASP Reference: https://owasp.org/Top10/A01_2021-Broken_Access_Control/
  CWE Reference: https://cwe.mitre.org/data/definitions/639.html

  ---
  Key Takeaway

  ▎ If the URL contains a user-controlled value that determines which account data is returned, and the server does not
  ▎ verify that value against the active session, every account on the platform is one parameter change away from being
  ▎ exposed. Always test account endpoints by substituting another user's identifier — if the server returns their data,
  ▎ authorization is missing entirely.
