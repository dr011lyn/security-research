  Lab Report: Multi-Step Process with No Access Control on One Step                                                        
                                                                                                                         
  Platform: PortSwigger Web Security Academy                                                                               
  Category: Broken Access Control                                                                                          
  Difficulty: Medium                                                                                                       
  Date Solved: 01 May 2026                                                                                                 
  Lab URL: https://portswigger.net/web-security/access-control/lab-multi-step-process-with-no-access-control-on-one-step

  ---
  Vulnerability Summary

  The application implements a two-step process for admin role assignment — step one selects the action, step two confirms
  it. The developer applied access control on step one but assumed that anyone reaching step two must have already passed
  step one legitimately. That assumption is wrong. Step two has no independent authorization check, meaning any
  authenticated user who sends the correct confirmation request — regardless of their role — can execute a privileged
  action. By capturing the admin's confirmation request and substituting a normal user's session cookie, an attacker can
  promote their own account to admin with no admin credentials.

  OWASP Category: A01:2021 — Broken Access Control
  CWE: CWE-284 (Improper Access Control) / CWE-602 (Client-Side Enforcement of Server-Side Security)

  ---
  How Serious Is It?

  Very serious — and extremely common in real applications. Multi-step workflows (checkout flows, approval chains, admin
  wizards) are one of the most frequently misconfigured access control patterns. Developers secure the entry point and
  forget every step after it. An attacker does not need to go through step one — they can craft step two directly. The more
   steps a workflow has, the more likely one of them is unprotected.

  ---
  Discovery

  Reconnaissance steps taken:

  1. Logged in as administrator — observed role management dropdown for upgrading and downgrading users
  2. Logged in as normal user (wiener) in a separate session — captured the session cookie in Burp Repeater
  3. Switched back to admin session — triggered a role upgrade action and intercepted the multi-step confirmation request
  in Burp Suite
  4. Identified that the confirmation request (step two) contained the target username and action in the request body
  5. Replaced the admin session cookie with the normal user's session cookie in the confirmation request
  6. Changed the target username to the account to be promoted
  7. Forwarded the request — server processed it successfully, role upgraded — lab solved

  What indicated something was wrong:

  The confirmation step was a standalone HTTP request with no session validation tied to admin privilege. The server
  trusted that if the confirmation request arrived with the correct parameters, the user must have been authorized at an
  earlier step. This assumption is the vulnerability — HTTP requests are stateless and can be crafted independently of any
  prior step.

  ---
  Exploitation

  Step 1 — Capture normal user session

  Logged in as wiener, sent the account page request to Burp Repeater, noted the session cookie.

  Step 2 — Observe the admin workflow

  Logged in as administrator, navigated to user management, triggered an upgrade action and intercepted both steps of the
  process in Burp Suite.

  Step 3 — Identify the unprotected confirmation step

  The confirmation request (step two) looked like:

  POST /admin-roles HTTP/2
  Host: [lab-host].web-security-academy.net
  Cookie: session=[admin-session-cookie]
  Content-Type: application/x-www-form-urlencoded

  username=carlos&action=upgrade&confirmed=true

  Step 4 — Substitute normal user session

  Replaced the admin session cookie with wiener's session cookie. Changed the target username to the account being
  promoted. Forwarded the request:

  POST /admin-roles HTTP/2
  Host: [lab-host].web-security-academy.net
  Cookie: session=[wiener-session-cookie]
  Content-Type: application/x-www-form-urlencoded

  username=wiener&action=upgrade&confirmed=true

  Server returned success — wiener promoted to admin. Lab solved.

  ---
  Impact Analysis

  What an attacker can do:
  - Bypass access control on any unprotected step in a multi-step workflow
  - Promote their own account to admin using only a normal user session
  - Execute any privileged action the confirmation step performs — role changes, deletions, approvals, financial
  transactions
  - The attack requires no admin credentials and leaves a minimal trace — the request looks like a normal form submission

  Who is affected:
  - Users: All accounts at risk once attacker holds admin role
  - Business: Any multi-step privileged workflow is potentially exploitable if individual steps are not independently
  authorized
  - Developers: The false confidence of securing step one means this class of bug is often missed entirely in code review

  Worst-case real-world scenario:
  A financial application uses a multi-step process to approve large fund transfers — step one requires manager approval,
  step two is the confirmation. An attacker discovers step two has no authorization check, crafts the confirmation request
  directly with their own session, and approves transfers without any manager involvement. The transaction logs show a
  valid confirmation request — no anomaly is visible until the money is gone.

  ---
  Root Cause

  The developer secured the workflow entry point and assumed that reaching a later step implied authorization at all prior
  steps. This is a fundamentally flawed security model — HTTP has no memory. Each request is independent. A user can send
  any request at any time regardless of what steps they have or have not completed. Every step in a privileged workflow
  must independently verify authorization as if it were the only step.

  ❌ Vulnerable assumption:
  Step 1 — check admin role ✅
  Step 2 — skip check, user must have passed step 1

  ✅ Correct approach:
  Step 1 — check admin role ✅
  Step 2 — check admin role ✅
  Step N — check admin role ✅

  ---
  Recommended Fix

  1. Apply authorization checks independently on every step — each request in a multi-step workflow must verify the
  requesting user's role server-side, with no assumption about prior steps
  2. Never trust workflow state on the client side — the server cannot assume a request for step two means step one was
  completed by an authorized user
  3. Use server-side state for multi-step flows — if step completion must be tracked, store it in the server-side session,
  not in the request parameters
  4. Audit every endpoint in a workflow individually — security reviews must test each step in isolation, not just the
  entry point

  OWASP Reference: https://owasp.org/Top10/A01_2021-Broken_Access_Control/
  CWE Reference: https://cwe.mitre.org/data/definitions/284.html

  ---
  Key Takeaway

  ▎ Securing step one of a multi-step process is not enough. An attacker does not follow your workflow — they send HTTP
  ▎ requests. Every step that performs a privileged action must independently verify the requesting user's authorization,
  ▎ with no assumption that a prior check already happened. Test each step of every workflow in isolation by sending the
  ▎ request directly, skipping all previous steps entirely.
