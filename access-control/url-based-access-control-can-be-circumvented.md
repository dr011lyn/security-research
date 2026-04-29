  Lab Report: URL-Based Access Control Can Be Circumvented
                                                                                          
  Platform: PortSwigger Web Security Academy
  Category: Broken Access Control                                                                                          
  Difficulty: High
  Date Solved: 28 April 2026                                                                                               
  Lab URL: https://portswigger.net/web-security/access-control/lab-url-based-access-control-can-be-circumvented

  ---
  Vulnerability Summary

  The application enforces access control on the front-end by blocking requests to restricted URLs like /admin. However,
  the back-end framework supports the X-Original-URL header — a non-standard HTTP header that instructs the server to route
   the request to a different URL than the one in the GET line. Because the front-end restriction only checks the URL in
  the GET request and not the header, an attacker can send GET / with X-Original-URL: /admin and bypass the restriction
  entirely — the front-end sees a harmless / request while the back-end processes /admin.

  OWASP Category: A01:2021 — Broken Access Control
  CWE: CWE-284 (Improper Access Control) / CWE-602 (Client-Side Enforcement of Server-Side Security)

  ---
  How Serious Is It?

  Extremely serious. This vulnerability means the entire access control layer is an illusion — it only blocks attackers who
   don't know about override headers. Any attacker familiar with X-Original-URL or X-Forwarded-URL headers can walk
  straight past it. The danger is that this pattern is common in applications that use a front-end proxy or WAF to enforce
  restrictions without corresponding server-side checks.

  ---
  Discovery

  Reconnaissance steps taken:

  1. Logged in with provided credentials and browsed the application
  2. Attempted to access /admin directly — received access denied
  3. Read the lab hint — back-end framework supports X-Original-URL header
  4. Intercepted the /admin request in Burp Suite and tested header injection

  What indicated something was wrong:

  The access control was enforced purely at the URL routing level with no server-side authorization check behind it. When
  the back-end framework processes X-Original-URL before the access control layer evaluates the request, the restriction
  becomes bypassable. The hint about the framework was the signal — but in a real target, testing override headers is
  standard methodology whenever URL-based blocking is observed.

  ---
  Exploitation

  1. Logged in with provided credentials — nothing unusual on the surface
  2. Attempted GET /admin — blocked with access denied
  3. Intercepted the request in Burp Suite
  4. Changed the GET line from GET /admin to GET / — removing the blocked path from the URL
  5. Added the header X-Original-URL: /admin to the request
  6. Forwarded the request — admin panel loaded successfully, all users visible
  7. Attempted to delete a user — access denied again on the delete action
  8. Intercepted the delete request, changed GET /admin/delete?username=wiener to GET /?username=wiener
  9. Added X-Original-URL: /admin/delete as a header — keeping the query parameter in the GET line
  10. Forwarded the request — user deleted successfully, lab solved

  Key technical detail: The query parameter (?username=wiener) must stay in the GET line, not the header. The header only
  overrides the path — the server reads query parameters from the original request line.

  Result: Full admin panel access and user deletion achieved by routing requests through X-Original-URL header, bypassing
  all URL-based restrictions.

  ---
  Impact Analysis

  What an attacker can do:
  - Bypass all URL-based access controls silently — no credentials needed beyond a regular user account
  - Access any restricted endpoint the back-end framework routes via X-Original-URL
  - Perform any admin action — view users, delete accounts, access sensitive data
  - The attack is invisible to front-end logs that only record the GET path — they show GET /, not GET /admin

  Who is affected:
  - Users: Full exposure — accounts can be viewed and deleted
  - Business: Any restricted functionality is effectively public to anyone who knows this technique
  - Security team: Access logs give a false sense of security since the restricted path never appears in the log

  Worst-case real-world scenario:
  An attacker discovers the application uses URL-based restrictions enforced by a front-end proxy. Using X-Original-URL,
  they silently access admin endpoints, enumerate all users, exfiltrate sensitive data, and make targeted deletions — while
   access logs show only harmless GET / requests. The breach goes undetected until business impact surfaces.

  ---
  Root Cause

  Access control was implemented at the URL routing layer only, without any authorization check on the server side behind
  it. The back-end framework honored X-Original-URL as a legitimate routing instruction, allowing the attacker to decouple
  what the front-end sees from what the back-end actually processes.

  Security through routing is not access control.

  ---
  Recommended Fix

  1. Enforce authorization server-side on every request — every sensitive endpoint must verify the authenticated user's
  role independently, regardless of how the request was routed
  2. Disable X-Original-URL and X-Forwarded-URL header processing if the application has no legitimate need for them — most
   frameworks allow this to be turned off
  3. Never rely solely on URL-based blocking — a WAF or front-end proxy blocking a URL path is a defense-in-depth layer,
  not a primary access control mechanism
  4. Audit framework defaults — know which non-standard headers your framework honors and disable any that are not required

  OWASP Reference: https://owasp.org/Top10/A01_2021-Broken_Access_Control/
  CWE Reference: https://cwe.mitre.org/data/definitions/602.html

  ---
  Key Takeaway

  ▎ URL-based access control enforced only at the routing layer is trivially bypassable. Whenever you see a 403 on a
  ▎ restricted path, test X-Original-URL, X-Forwarded-URL, and path override headers before giving up. The front-end
  ▎ blocking a URL means nothing if the back-end will still process it when asked through a header. Real access control
  ▎ lives in the business logic, not the router.
