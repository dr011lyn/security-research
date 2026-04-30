  Lab Report: Method-Based Access Control Can Be Circumvented
                                                                                           
  Platform: PortSwigger Web Security Academy
  Category: Broken Access Control                                                                                          
  Difficulty: High
  Date Solved: 30 April 2026                                                                                               
  Lab URL: https://portswigger.net/web-security/access-control/lab-method-based-access-control-can-be-circumvented

  ---
  Vulnerability Summary

  The application enforces access control on the role-assignment endpoint /admin-roles only for the POST method. When the
  HTTP method is changed to a non-standard value such as POSTX, the authorization check is skipped entirely and the server
  proceeds to process the request. By intercepting the admin's role-change request, substituting a normal user's session
  cookie, and changing POST to POSTX, an attacker can promote their own account to admin with no admin credentials
  required.

  OWASP Category: A01:2021 — Broken Access Control
  CWE: CWE-284 (Improper Access Control) / CWE-436 (Interpretation Conflict)

  ---
  How Serious Is It?

  Critical. Any authenticated regular user can escalate themselves to admin by modifying a single word in an HTTP request.
  No special knowledge beyond basic Burp Suite usage is required. In production, this is particularly dangerous because the
   attack uses a valid session and touches a real endpoint — it blends into normal traffic and is unlikely to trigger
  alerts.

  ---
  Discovery

  Reconnaissance steps taken:

  1. Logged in as admin — observed role assignment dropdown on the admin panel
  2. Assigned a role to a user and intercepted the request in Burp Suite — captured the POST /admin-roles request
  3. Logged in as normal user (wiener) in a separate session — captured the session cookie from the GET
  /my-account?id=wiener request
  4. In the admin role-change request, replaced the admin session cookie with the normal user session cookie — received 401
   Unauthorized
  5. Changed the HTTP method from POST to POSTX — received 400 Bad Request: "Missing parameter 'username'" instead of 401
  6. Recognized the missing parameter error as confirmation the auth check was bypassed — added correct parameters and
  forwarded

  What indicated something was wrong:

  The change from 401 Unauthorized to 400 Bad Request when switching methods was the critical signal. 401 means the server
  checked authorization and blocked it. 400 means the server moved past authorization entirely and started processing the
  request — it just needed the right parameters. The access control was method-specific.

  ---
  Exploitation

  Step 1 — Capture normal user session

  Logged in as wiener and captured the session cookie from the account page request:

  GET /my-account?id=wiener HTTP/2
  Host: 0ab300bf03eddff881d5a29f009400a1.web-security-academy.net
  Cookie: session=8luPxkZrLKnaRVCcfWfMiGOO5jkdpOTo

  Step 2 — Intercept admin role-change request

  Logged in as admin, triggered the role upgrade action, intercepted the POST /admin-roles request in Burp Suite and sent
  it to Repeater.

  Step 3 — Swap session cookie

  Replaced the admin session cookie with wiener's session cookie. Forwarded — received 401 Unauthorized. Authorization
  check was working for POST.

  Step 4 — Change HTTP method to POSTX

  Changed POST to POSTX. Forwarded the request:

  POSTX /admin-roles HTTP/2
  Host: 0ab300bf03eddff881d5a29f009400a1.web-security-academy.net
  Cookie: session=8luPxkZrLKnaRVCcfWfMiGOO5jkdpOTo
  Content-Type: application/x-www-form-urlencoded

  username=carlos&action=upgrade

  Server response:

  HTTP/2 400 Bad Request
  Content-Type: application/json; charset=utf-8

  "Missing parameter 'username'"

  Authorization bypassed. Server was now processing the request but not finding the username parameter correctly due to the
   non-standard method.

  Step 5 — Fix parameters and forward

  Adjusted the parameter format to match what the server expected with a non-standard method. Added
  username=carlos&action=upgrade correctly — server accepted the request, promoted carlos to admin. Lab solved.

  ---
  Impact Analysis

  What an attacker can do:
  - Escalate any regular user account to admin using nothing but a valid session cookie and Burp Suite
  - Access full admin functionality — view, delete, and manage all user accounts
  - Create persistent backdoor admin accounts for long-term access
  - Operate under a legitimate user session — harder to distinguish from normal traffic in logs

  Who is affected:
  - Users: All accounts exposed once attacker holds admin role
  - Business: The entire privilege model is broken — any user can become admin
  - Admins: Their exclusive access is rendered meaningless
  - Security team: Attack uses a valid session, making detection difficult without method-level logging

  Worst-case real-world scenario:
  An attacker with a free registered account intercepts one request, changes one word, and becomes admin. They promote a
  second throwaway account to admin as a backdoor, demote or delete existing admins, and maintain persistent privileged
  access — while the application logs show nothing but normal-looking requests from a valid user session.

  ---
  Root Cause

  The server implemented the authorization check inside the POST handler for /admin-roles. When the method was POST, the
  check ran. When the method was anything else, the check was skipped and the request fell through directly to business
  logic. Access control must be enforced at the endpoint level — wrapping all methods — not inside one specific method
  handler.

  ---
  Recommended Fix

  1. Enforce authorization on the endpoint, not the method — the /admin-roles endpoint must require admin session for every
   HTTP method, including non-standard ones like POSTX
  2. Reject non-standard HTTP methods at middleware level — return 405 Method Not Allowed for any method not explicitly
  supported before business logic runs
  3. Never place access control checks inside method-specific handlers — use middleware or decorators that wrap the entire
  route
  4. Include method-switching in security testing checklists — every privileged endpoint should be tested with GET, PUT,
  PATCH, HEAD, and arbitrary methods during review

  OWASP Reference: https://owasp.org/Top10/A01_2021-Broken_Access_Control/
  CWE Reference: https://cwe.mitre.org/data/definitions/284.html

  ---
  Key Takeaway

  ▎ A 401 changing to a 400 when you switch HTTP methods is one of the clearest signals in web security — the auth check
  ▎ just disappeared. If changing POST to POSTX bypasses authorization, the check was never protecting the endpoint, it was
  ▎  protecting one specific way of reaching it. Always test privileged endpoints with non-standard methods. The server's
  ▎ own error messages will tell you whether it's processing your request or blocking it.
