  Lab Report: Unprotected Admin Functionality with Unpredictable URL

  Platform: PortSwigger Web Security Academy
  Category: Broken Access Control
  Difficulty: Low
  Date Solved: 27 April 2026
  Lab URL: https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality-with-unpredictable-url

  ---
  Vulnerability Summary

  The application exposes a hidden admin panel via a URL containing a random/unpredictable string, with the intent that
  obscurity alone would prevent unauthorized access. However, this URL was hardcoded and leaked directly in the
  application's client-side source code, allowing any attacker who inspects the page to discover and access the panel
  without any authentication or authorization check.

  OWASP Category: A01:2021 — Broken Access Control
  CWE: CWE-284 (Improper Access Control) / CWE-200 (Exposure of Sensitive Information)

  ---
  Discovery

  Reconnaissance steps taken:

  1. Checked robots.txt — no admin path disclosed there
  2. Reviewed JavaScript files for embedded endpoints
  3. Inspected the page source and searched for the keyword admin
  4. Found an href attribute pointing to an admin panel URL that included a random string (security-through-obscurity)

  What indicated something was wrong:

  A hardcoded <a href="/admin-<randomstring>"> was present in the HTML source, revealing the exact path to the admin panel.
   The path was intentionally non-guessable but was inadvertently published in the client-facing code.

  ---
  
  Exploitation
                                                                                                                           
  1. Opened the lab and browsed the application
  2. Checked `robots.txt` — no findings                                                                                    
  3. Inspected page source, searched for `admin`
  4. Discovered hardcoded `admin href` with random path segment
  5. Appended the path to the base URL and navigated directly
  6. Admin panel loaded with no authentication prompt
  7. Used admin functionality to delete users — lab solved

  **Result:** Full admin panel access achieved. User deletion confirmed, lab solved.

  Proof of Success: Full admin panel access achieved. User deletion functionality executed successfully, confirming
  unrestricted administrative access.

  ---
  Impact Analysis

  What an attacker can do:
  - Access all admin panel functionality without any credentials
  - Create or delete user accounts
  - Potentially access sensitive user data visible only to admins
  - Perform destructive actions (bulk deletion, data modification) under the guise of a legitimate admin

  Who is affected:
  - Business: Unauthorized party gains full administrative control
  - Users: Their accounts and data are at risk of deletion or exposure
  - Admins: Their privileged access is effectively bypassed

  Worst-case real-world scenario:
  An attacker discovers the URL, silently monitors admin activity, then — at a critical moment — deletes all user accounts,
   cancels active contracts, or exfiltrates client data. Because no auth log ties an account to the action, attribution is
  difficult. The business suffers reputational and financial damage with limited forensic trail.

  ---
  Root Cause

  The developer relied on security through obscurity — assuming that a random URL string is sufficient protection. This
  assumption fails because:

  1. The URL was exposed in client-side source code (accessible to anyone with a browser)
  2. No server-side authorization check verified whether the requesting user was actually an admin

  Obscurity is never a substitute for access control.

  ---
  Recommended Fix

  1. Implement server-side authorization on all admin routes — verify the authenticated user's role before serving any
  admin page or processing admin actions
  2. Never embed sensitive paths in client-side code — admin URLs should not appear in HTML, JavaScript, or any resource
  delivered to unauthenticated users
  3. Apply defense in depth — unpredictable URLs can be a supplementary measure but must never be the only protection

  ---
  Key Takeaway

  ▎ Source code review is a first-class recon technique. Developers sometimes hide sensitive infrastructure behind obscure
  ▎ URLs but forget that the client receives everything needed to find them. In a real bug bounty engagement, always grep
  ▎ the source for keywords like admin, panel, internal, debug, and secret before moving on.
