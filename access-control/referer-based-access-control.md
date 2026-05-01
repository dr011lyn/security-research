  Lab Report: Referer-Based Access Control     

  Platform: PortSwigger Web Security Academy                                                                               
  Category: Broken Access Control
  Difficulty: Medium                                                                                                       
  Date Solved: 01 May 2026
  Lab URL: https://portswigger.net/web-security/access-control/lab-referer-based-access-control

  ---
  Vulnerability Summary

  The application uses the HTTP Referer header to control access to the admin role-assignment endpoint. If a request to
  /admin-roles arrives with Referer: /admin, it is allowed. If the header is absent or points elsewhere, it is rejected. An
   attacker who captures any legitimate admin request to that endpoint already has a request with the correct Referer
  header. By substituting their own session cookie into that captured request, they can execute privileged actions — the
  server checks where the request claims to come from, not who is actually sending it.

  The Referer header is client-controlled. It can be set to any value by anyone.

  OWASP Category: A01:2021 — Broken Access Control
  CWE: CWE-284 (Improper Access Control) / CWE-807 (Reliance on Untrusted Inputs in a Security Decision)

  ---
  How Serious Is It?

  Very serious. This is a complete authorization bypass using a single HTTP header that any client can forge. The Referer
  header was designed to tell servers where a request originated — it was never intended as a security control. Any
  developer using it to gate privileged functionality has built access control on a foundation the attacker controls
  entirely. The bypass requires no special tools — just Burp Suite and a valid user session.

  ---
  Discovery

  Reconnaissance steps taken:

  1. Logged in as administrator — observed role management dropdown for upgrading and downgrading users
  2. Intercepted the role upgrade request in Burp Suite — noted the Referer header pointing to /admin
  3. Sent the request to Repeater
  4. Logged in as normal user (wiener) in a separate incognito session — captured the session cookie
  5. Attempted to access /admin-roles?username=carlos&action=upgrade directly as the normal user — received unauthorized
  response due to missing Referer header
  6. Substituted the normal user's session cookie into the captured admin request in Repeater — kept the Referer: /admin
  header intact
  7. Changed the target username — forwarded the request — server accepted it, role upgraded — lab solved

  What indicated something was wrong:

  When the direct request without a Referer header was rejected but the same request with Referer: /admin was accepted
  regardless of the session cookie, the access control mechanism was clearly tied to the header rather than the user's
  actual role. The Referer header is entirely client-controlled — any attacker with a captured request already has
  everything they need to forge it.

  ---
  Exploitation

  Step 1 — Capture admin role-change request

  Logged in as administrator. Triggered a role upgrade and intercepted the request in Burp Suite:

  GET /admin-roles?username=carlos&action=upgrade HTTP/2
  Host: [lab-host].web-security-academy.net
  Cookie: session=[admin-session-cookie]
  Referer: https://[lab-host].web-security-academy.net/admin

  Sent to Repeater.

  Step 2 — Capture normal user session

  Logged in as wiener in an incognito window. Noted the session cookie.

  Step 3 — Confirm the Referer dependency

  Accessed /admin-roles?username=carlos&action=upgrade directly as wiener with no Referer header — server returned
  unauthorized.

  Step 4 — Forge the request

  In Repeater, replaced the admin session cookie with wiener's session cookie. Kept the Referer: /admin header unchanged.
  Changed the target username to wiener:

  GET /admin-roles?username=wiener&action=upgrade HTTP/2
  Host: [lab-host].web-security-academy.net
  Cookie: session=[wiener-session-cookie]
  Referer: https://[lab-host].web-security-academy.net/admin

  Step 5 — Forward

  Server accepted the request — wiener promoted to admin. Lab solved.

  ---
  Impact Analysis

  What an attacker can do:
  - Bypass all access control on any endpoint protected only by Referer validation
  - Promote their own account to admin using a normal user session and one captured request
  - Execute any privileged action the endpoint performs — role changes, deletions, configuration updates
  - The attack is nearly invisible — the request looks structurally identical to a legitimate admin request

  Who is affected:
  - Users: All accounts exposed once attacker holds admin role
  - Business: Every endpoint using Referer-based access control is trivially bypassable by any authenticated user
  - Security team: Logs show a valid-looking request with the correct Referer — no anomaly unless session roles are being
  monitored

  Worst-case real-world scenario:
  An application protects its entire admin section by checking that requests come from within the admin panel using
  Referer. An attacker captures one admin request through any means — a shared network, a leaked log, or just by observing
  their own admin session on a test account — and uses it as a template to execute any admin action with their own session.
   Every privileged endpoint in the application is now accessible to any authenticated user who knows this technique.

  ---
  Root Cause

  The developer used the Referer header as a proxy for authorization — assuming that only someone on the admin panel could
  generate a request with Referer: /admin. This assumption fails because:

  1. The Referer header is set by the client — any HTTP client including Burp Suite can set it to any value
  2. Headers are not credentials — they carry no cryptographic proof of origin
  3. Authorization must be tied to identity — only server-side session data can reliably establish who is making a request
  and what they are allowed to do

  ---
  Recommended Fix

  1. Enforce role-based authorization server-side on every request — every call to /admin-roles must verify that the
  session belongs to an admin, regardless of what headers accompany the request
  2. Never use Referer as a security control — it is informational only, fully client-controlled, and frequently absent due
   to privacy settings, browser behavior, and HTTPS stripping
  3. Treat all HTTP headers as untrusted input — Referer, X-Forwarded-For, Origin, and similar headers can all be forged.
  Security decisions must never depend on them alone
  4. Apply consistent authorization middleware — all admin endpoints should share a single server-side role check that
  cannot be bypassed by manipulating request metadata

  OWASP Reference: https://owasp.org/Top10/A01_2021-Broken_Access_Control/
  CWE Reference: https://cwe.mitre.org/data/definitions/807.html

  ---
  Key Takeaway

  ▎ The Referer header tells the server where a request claims to come from. The word "claims" is the problem — the client
  ▎ sets it, which means the attacker sets it. Any access control that can be bypassed by adding or changing a single HTTP
  ▎ header is not access control at all. Authorization must always be derived from server-side session state, never from
  ▎ values the client sends in the request.
