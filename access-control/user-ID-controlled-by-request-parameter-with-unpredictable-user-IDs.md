  Lab Report: User ID Controlled by Request Parameter with Unpredictable User IDs                                          
                                                                                                                         
  Platform: PortSwigger Web Security Academy                                                                               
  Category: Broken Access Control — Insecure Direct Object Reference (IDOR)                                                
  Difficulty: Medium                                                                                                       
  Date Solved: 30 April 2026                                                                                               
  Lab URL: https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter-with-unpredictab
  le-user-ids

  ---
  Vulnerability Summary

  The application uses GUIDs as user identifiers — long unpredictable strings intended to prevent enumeration. However, the
   same GUIDs are leaked in publicly accessible blog post pages, embedded in author links in the HTML response. An attacker
   who cannot guess the GUID can simply find it by browsing the application's public content. Once obtained, the GUID can
  be used in the account endpoint to access the target user's data with no authorization error — the server never verifies
  that the requesting session owns the account being accessed.

  GUIDs add friction but provide no real security. This is still a textbook Insecure Direct Object Reference (IDOR).

  OWASP Category: A01:2021 — Broken Access Control
  CWE: CWE-639 (Authorization Bypass Through User-Controlled Key) / CWE-284 (Improper Access Control)

  ---
  How Serious Is It?

  Serious — and more common in real applications than the simpler version. Many developers believe switching from
  sequential IDs to GUIDs fixes IDOR. It does not. If the GUID appears anywhere in public-facing content — blog posts,
  comments, public profiles, API responses, HTML source — it is no longer secret. An attacker only needs to find one leak
  to exploit the vulnerability. The false sense of security GUIDs create can actually make this worse, because developers
  stop looking for the authorization problem.

  ---
  Discovery

  Reconnaissance steps taken:

  1. Logged in with provided credentials — observed GUID assigned to own account in traffic
  2. Checked page source, JavaScript files, and URLs — could not find the target user's GUID directly
  3. Shifted focus to public-facing features — navigated to the blog section before login
  4. Opened blog posts one by one and monitored traffic in Burp Suite
  5. On the second post — authored by the target user — inspected the HTTP response
  6. Found the target user's GUID leaked in the HTML as an author link:
  <a href='/blogs?userId=cfb7b407-6d4d-4f2e-9a17-36a75f7448ec'>
  7. Used the extracted GUID to access the target's account page — full access granted

  What indicated something was wrong:

  The GUID assigned to the logged-in user appeared in the account URL as a parameter the client controlled. This is the
  same IDOR pattern as sequential IDs — the only difference is the ID format. Once the target's GUID was found in the blog
  response, the vulnerability was identical to a simple username-based IDOR.

  ---
  Exploitation

  Step 1 — Identify the ID format

  Logged in as wiener, intercepted traffic in Burp Suite. Observed account URL used a GUID parameter — a long non-guessable
   string.

  Step 2 — Attempt direct discovery

  Checked source code, JavaScript files, and all URLs for the target user's GUID — no findings.

  Step 3 — Find the GUID leak in blog posts

  Navigated to the public blog. Intercepted the request for the second post:

  GET /post?postId=9 HTTP/2
  Host: 0ac4003b034ede2780f158a500f8008a.web-security-academy.net
  Cookie: session=j8OBwbBkdUL0Qeek2LFa7CfakSL4cqAS

  Inspected the HTML response — found the target user's GUID embedded in the author link:

  <a href='/blogs?userId=cfb7b407-6d4d-4f2e-9a17-36a75f7448ec'>

  Step 4 — Access target account

  Used the extracted GUID in the account endpoint:

  GET /my-account?id=cfb7b407-6d4d-4f2e-9a17-36a75f7448ec HTTP/2
  Cookie: session=j8OBwbBkdUL0Qeek2LFa7CfakSL4cqAS

  Server returned the target user's account page including their API key — no authorization error.

  Step 5 — Submit API key

  Copied the target user's API key from the response, submitted it — lab solved.

  ---
  Impact Analysis

  What an attacker can do:
  - Harvest GUIDs from public content (blog posts, comments, public profiles) and use them to access private account data
  - Access API keys, personal information, and any data on the account page
  - Scale the attack by scraping all public content for GUIDs, then systematically accessing every account found
  - Use stolen API keys to authenticate as other users in connected services

  Who is affected:
  - Users: Every user who has authored public content has their GUID exposed — their private account data is accessible
  - Business: Mass account data exposure through a feature the developer believed was secure
  - Compliance: API key exposure could lead to unauthorized access across integrated services — significant breach
  liability

  Worst-case real-world scenario:
  An attacker scrapes all public blog posts, comments, and profiles to collect GUIDs for every user who has ever posted
  publicly. They then systematically access each user's private account page, harvesting API keys, personal data, and
  billing information. The server logs show only normal authenticated requests — nothing stands out because each request
  uses a valid session and a valid GUID. The breach only surfaces when stolen credentials are used elsewhere.

  ---
  Root Cause

  Two compounding issues created this vulnerability:

  1. No server-side ownership check — the account endpoint returned any user's data when their GUID was provided,
  regardless of whether the requesting session owned that account
  2. GUID leaked in public content — the application exposed user GUIDs in HTML responses for blog posts, assuming they
  were safe because they were non-guessable

  Unpredictable IDs only add security if they are never exposed. The moment a GUID appears in any public response, it
  becomes as useful to an attacker as a sequential integer.

  ---
  Recommended Fix

  1. Enforce server-side ownership checks — the account endpoint must verify that the GUID in the request matches the user
  tied to the active session, regardless of how unpredictable the GUID is
  2. Never expose internal user IDs in public-facing content — author links, public profiles, and any public HTML should
  use display names or separate public identifiers, never the same ID used to access private account data
  3. Treat GUIDs as defense-in-depth only — unpredictable IDs make enumeration harder but are not a substitute for
  authorization. The authorization check must exist independently
  4. Audit all places where user IDs appear in responses — search the entire codebase for every location where a user's
  account ID is included in an HTTP response and assess whether that context is public

  OWASP Reference: https://owasp.org/Top10/A01_2021-Broken_Access_Control/
  CWE Reference: https://cwe.mitre.org/data/definitions/639.html

  ---
  Key Takeaway

  ▎ GUIDs are not access control. They are speed bumps — they slow down guessing but do nothing once the ID is known. In
  ▎ real applications, user IDs leak constantly: blog posts, comments, public profiles, API responses, email headers.
  ▎ Always check every public-facing feature for ID leaks before concluding that unpredictable IDs make an endpoint safe.
  ▎ The authorization check must exist regardless of how the ID was obtained.
