  Lab Report: User Role Can Be Modified in User Profile
                                                                                          
  Platform: PortSwigger Web Security Academy
  Category: Broken Access Control                                                                                          
  Difficulty: Medium
  Date Solved: 28 April 2026                                                                                               
  Lab URL: https://portswigger.net/web-security/access-control/lab-user-role-can-be-modified-in-user-profile

  ---
  Vulnerability Summary

  The application exposes a user profile update endpoint that accepts a JSON request to change email. The server processes
  the request and returns a JSON response containing user properties including role. The vulnerability is that the server
  also accepts and applies additional JSON fields — including roleid — that were never intended to be user-controlled. By
  injecting "roleid": 2 into the update request, an attacker silently escalates their privileges to admin without any error
   or rejection from the server.

  This is a Mass Assignment vulnerability combined with broken access control — the server blindly applies all properties
  it receives rather than whitelisting only the ones a regular user is allowed to change.

  OWASP Category: A01:2021 — Broken Access Control
  CWE: CWE-915 (Improperly Controlled Modification of Dynamically-Determined Object Attributes) / CWE-284 (Improper Access
  Control)

  ---
  Discovery

  Reconnaissance steps taken:

  1. Logged in with provided credentials
  2. Checked robots.txt, sitemap.xml, and page source — no direct findings
  3. Identified an Update Email functionality on the user profile page
  4. Submitted an email update and intercepted the request with Burp Suite
  5. Observed the request was sending JSON and the response returned a full user object including username, email, and
  apikey
  6. Injected "roleid": 2 into the request JSON — server accepted it without error
  7. Response confirmed role elevation — admin panel became accessible

  What indicated something was wrong:

  The server response returned more user properties than it should have exposed — including internal fields like apikey and
   role-related data. When a server returns internal object properties to the client, it often means those same properties
  can be written back. That mismatch between what the server exposes and what it should expose was the signal.

  Attempts before finding it: The update email function appeared normal at first — the vulnerability only became visible
  after intercepting and inspecting the full JSON exchange.

  ---
  Exploitation

  1. Opened the lab, logged in with provided credentials
  2. Checked robots.txt, sitemap.xml, and page source — no findings
  3. Navigated to the Update Email functionality on the profile page
  4. Submitted an email update with Burp Suite intercepting the request
  5. Observed request body: {"email": "test@test.com"}
  6. Observed response body: JSON object containing username, email, apikey, and role data
  7. Modified the request to inject "roleid": 2 alongside the email field
  8. Server accepted the request — no error, no rejection
  9. Response confirmed role updated to admin
  10. Navigated to admin panel — full access granted, all users visible
  11. Deleted the target user — lab solved

  Result: Privilege escalation from regular user to admin achieved by injecting one field into a JSON request. Full admin
  panel access confirmed.

  ---
  Impact Analysis

  What an attacker can do:
  - Escalate privileges from any regular user account to admin silently
  - Access all admin functionality — view, create, delete user accounts
  - No special tools required — only Burp Suite and a valid user account
  - The attack leaves no obvious trace since the request looks like a normal profile update

  Who is affected:
  - Users: All accounts at risk of deletion or data exposure
  - Business: Any registered user can become a self-appointed admin
  - Admins: Their exclusive access is completely undermined

  Worst-case real-world scenario:
  An attacker registers a free account, intercepts the profile update request, injects roleid:2, and gains permanent admin
  access. From there they exfiltrate all user data, modify or delete records, and potentially create backdoor admin
  accounts for persistent access — all while appearing to be a legitimate user in the logs.

  ---
  Root Cause

  The server uses a pattern where it takes the incoming JSON object and maps it directly onto the user model without
  filtering which fields are allowed to be updated. This means any property that exists on the user object — including
  privileged ones like roleid — can be overwritten by the client simply by including it in the request.

  The server should maintain a strict allowlist of fields a user is permitted to update. Everything outside that list
  should be ignored or rejected.

  ---
  Recommended Fix

  1. Whitelist accepted fields server-side — the email update endpoint should only process email, nothing else. Any
  additional fields in the request should be stripped and ignored
  2. Never expose internal object properties to the client — the response returning apikey, role data, and other internal
  fields is itself a vulnerability. Return only what the UI needs
  3. Enforce role changes through privileged endpoints only — role assignment should be a separate, admin-only operation
  with its own authorization check, never a side effect of a profile update
  4. Implement server-side privilege validation on every request — the server should derive the user's role from its own
  session store, not from anything in the request body

  OWASP Reference: https://owasp.org/Top10/A01_2021-Broken_Access_Control/
  CWE Reference: https://cwe.mitre.org/data/definitions/915.html

  ---
  Key Takeaway

  ▎ When a server responds with more data than the UI needs — internal IDs, API keys, role fields — test whether those same
  ▎  fields can be sent back in a write request. Servers that blindly map incoming JSON onto internal objects are
  ▎ vulnerable to mass assignment. If the server tells you a field exists, always check whether you can overwrite it.
