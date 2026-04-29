  Lab Report: User Role Controlled by Request Parameter
                                                                                                                
  Platform: PortSwigger Web Security Academy
  Category: Broken Access Control                                                                                          
  Difficulty: Medium
  Date Solved: 28 April 2026                                                                                               
  Lab URL: https://portswigger.net/web-security/access-control/lab-user-role-controlled-by-request-parameter

  ---
  Vulnerability Summary

  The application trusts a client-controlled cookie parameter (Admin=false/true) to determine whether a user has
  administrative privileges. Because this value is set on the client side and never validated on the server, any attacker
  who intercepts and modifies the request can escalate their privileges to admin level with no credentials or special
  access required.

  OWASP Category: A01:2021 — Broken Access Control
  CWE: CWE-639 (Authorization Bypass Through User-Controlled Key) / CWE-284 (Improper Access Control)

  ---
  Discovery

  Reconnaissance steps taken:

  1. Reviewed page source, robots.txt, and sitemap.xml — no direct findings
  2. Logged in using provided credentials with Burp Suite intercepting traffic
  3. Forwarded the intercepted login request and inspected the response
  4. Found Admin=false set as a cookie parameter in the server response

  What indicated something was wrong:

  The server was sending Admin=false in the cookie and trusting whatever value came back in subsequent requests.
  Authorization state should never be stored in a client-controlled value — the server should determine role from its own
  session store, not from what the browser sends.

  Attempts before finding it: 3

  ---
  Exploitation

  1. Opened the login page and started Burp Suite intercept
  2. Checked robots.txt, sitemap.xml, and page source — no findings
  3. Logged in with provided credentials, intercepted the login request
  4. Forwarded the request — response contained cookie Admin=false
  5. Modified the cookie value to Admin=true and forwarded
  6. Application granted full admin panel access
  7. Viewed all users and deleted the target user — lab solved

  Result: Full admin panel access achieved without any admin credentials. User deletion confirmed, lab solved.

  ---
  Impact Analysis

  What an attacker can do:
  - Instantly escalate from regular user to admin by modifying one cookie value
  - Access all admin functionality — view, create, and delete user accounts
  - Operate as admin with no audit trail tied to a legitimate admin account
  - Access sensitive data visible only to admins

  Who is affected:
  - Users: Accounts exposed to unauthorized deletion or data access
  - Business: Any unauthenticated attacker can silently assume admin control
  - Admins: Their privileged access is fully bypassed — they may not even know

  Worst-case real-world scenario:
  An attacker logs in as a regular user, flips the cookie, and gains persistent admin access. They exfiltrate all user
  data, cancel active client contracts, delete accounts, and exit — leaving the business with data breach liability,
  reputational damage, and no clear forensic trail pointing to how the access was obtained.

  ---
  Root Cause

  The developer stored authorization state (Admin=true/false) in a client-side cookie and trusted it on every subsequent
  request without server-side verification. This is a fundamental design flaw — the client should never be trusted to
  declare its own privilege level.

  The server must determine authorization from its own session data, not from values the browser sends back.

  ---
  Recommended Fix

  1. Never store authorization roles in client-side cookies — store only a session ID on the client, and keep the role
  mapped to that session on the server
  2. Validate every privileged request server-side — check the authenticated user's role from the server session before
  processing any admin action
  3. Sign and encrypt sensitive cookies — if any sensitive state must live in a cookie, use HMAC signing so tampering is
  detectable
  4. Implement least privilege — regular user sessions should have no path to admin functionality regardless of what
  parameters they send

  OWASP Reference: https://owasp.org/Top10/A01_2021-Broken_Access_Control/
  CWE Reference: https://cwe.mitre.org/data/definitions/284.html

  ---
  Key Takeaway

  ▎ Never trust the client to declare its own role. If the server sends Admin=false in a cookie and then honors Admin=true 
  ▎ when it comes back, the attacker only needs a browser and Burp Suite to become an admin. Authorization must always be
  ▎ enforced server-side, derived from session state the client cannot touch.
