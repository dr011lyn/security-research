  Lab Report: Unprotected Admin Functionality
                                                       
  Platform: PortSwigger Web Security Academy
  Category: Broken Access Control                                                                                          
  Difficulty: Low
  Date Solved: 26 April 2026                                                                                               
  Lab URL: https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality

  ---
  Vulnerability Summary

  The application exposes its admin panel URL inside robots.txt under the Disallow directive — a file intended to instruct
  search engine crawlers which paths to skip. The admin panel itself has zero authentication, meaning anyone who discovers
  the URL can access full administrative functionality with no credentials. The developer attempted to hide the panel by
  keeping it out of search results, but robots.txt is publicly readable by anyone — including attackers — making it a
  roadmap to sensitive endpoints rather than a protection mechanism.

  OWASP Category: A01:2021 — Broken Access Control
  CWE: CWE-284 (Improper Access Control) / CWE-200 (Exposure of Sensitive Information)

  ---
  Discovery

  Reconnaissance steps taken:

  1. Opened the target — standard shopping website, nothing unusual visible
  2. Inspected page source — no sensitive paths found
  3. Checked sitemap.xml — no findings
  4. Reviewed JavaScript files from source — nothing of interest
  5. Checked robots.txt — admin panel path found in the Disallow directive

  What indicated something was wrong:

  The robots.txt file contained a Disallow entry for the admin panel path. Developers add paths to Disallow specifically
  because they want to hide them — but robots.txt is a public file. Every path listed there is effectively being announced
  to anyone who checks it.

  ---
  Exploitation

  1. Opened the lab — observed a standard shopping website
  2. Checked source code, sitemap.xml, and JavaScript files — no findings
  3. Navigated to /robots.txt — found admin panel path listed under Disallow
  4. Appended the path to the base URL and navigated directly
  5. Admin panel loaded with no login page, no authentication prompt
  6. Full user list visible immediately
  7. Deleted the target user — lab solved

  Result: Admin panel accessed with no credentials. Full administrative functionality available to anyone with the URL.

  ---
  Impact Analysis

  What an attacker can do:
  - Access the admin panel instantly with no credentials
  - View, create, and delete all user accounts
  - Abuse any admin functionality the panel exposes
  - Use robots.txt as a recon tool to map out other hidden paths on the same target

  Who is affected:
  - Users: All accounts exposed to unauthorized access and deletion
  - Business: Complete loss of admin control to any anonymous attacker
  - Security team: No authentication means no audit trail — the attack is invisible

  Worst-case real-world scenario:
  An attacker checks robots.txt during routine recon, finds the admin path, opens it, and gains unrestricted admin access
  in under two minutes. They enumerate all users, exfiltrate customer data, delete accounts, and leave — with no login
  event recorded anywhere because no authentication ever occurred.

  ---
  Root Cause

  Two compounding mistakes created this vulnerability:

  1. The admin URL was listed in robots.txt — a public file, readable by anyone, that directly revealed the hidden path
  2. The admin panel had no authentication — once the URL was known, there was nothing stopping access

  Either mistake alone would be serious. Together they make the admin panel effectively public.

  ---
  Recommended Fix

  1. Implement authentication on all admin endpoints — every request to an admin path must verify the user is logged in and
   holds an admin role, server-side, on every request
  2. Never list sensitive paths in robots.txt — if a path needs to be hidden, hiding it in a public file defeats the
  purpose entirely. Use proper access control instead
  3. Treat robots.txt as public documentation — assume every path listed there will be visited by an attacker. Only list
  paths that are already protected

  OWASP Reference: https://owasp.org/Top10/A01_2021-Broken_Access_Control/
  CWE Reference: https://cwe.mitre.org/data/definitions/200.html

  ---
  Key Takeaway

  ▎ robots.txt is not a security mechanism — it is a public announcement. Every path you add to Disallow is a path you are
  ▎ telling every visitor, including attackers, to go look at. Always check robots.txt early in recon. And never ship an
  ▎ admin panel without authentication, no matter how obscure you think the URL is.
