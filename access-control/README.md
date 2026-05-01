  # Access Control Vulnerabilities                                                                                         
                                                                                                                           
  This folder contains detailed write-ups for every PortSwigger Web Security Academy lab                                   
  in the Access Control category, solved hands-on and documented with full exploitation                                    
  steps, impact analysis, and developer fix recommendations.                                                               

  All findings are mapped to **OWASP A01:2021 — Broken Access Control**.

  ---

  ## What is Broken Access Control?

  Access control enforces that users can only act within their intended permissions.
  When these controls fail, attackers can access unauthorized functionality or data —
  including admin panels, other users' accounts, or sensitive operations.

  It has ranked **#1 in the OWASP Top 10 since 2021**.

  ---

  ## Lab Progress

  | # | Lab | Difficulty | Status | Date |                                                                                 
  |---|-----|------------|--------|------|
  | 1 | Unprotected admin functionality | Low | ✅ Solved | 26-04-2026 |                                                   
  | 2 | Unprotected admin - unpredictable URL | Low | ✅ Solved | 27-04-2026 |
  | 3 | User role - request parameter | Medium | ✅ Solved | 28-04-2026 |
  | 4 | User role - modified in profile | Medium | ✅ Solved | 28-04-2026 |
  | 5 | URL-based access control bypass | High | ✅ Solved | 28-04-2026 |
  | 6 | Method-based access control bypass | High | ✅ Solved | 30-04-2026 |
  | 7 | User ID - request parameter | Low | ✅ Solved | 30-04-2026 |
  | 8 | User ID - unpredictable IDs | Medium | ✅ Solved | 30-04-2026 |
  | 9 | User ID - data leakage in redirect | Low | ✅ Solved | 01-05-2026 |
  | 10 | User ID - password disclosure | Medium | ✅ Solved | 01-05-2026 |
  | 11 | Multi-step - missing access control | Medium | ✅ Solved | 01-05-2026 |
  | 12 | Referer-based access control | Medium | ✅ Solved | 01-05-2026 |

  ---

  ## Resources

  - [PortSwigger Access Control Labs](https://portswigger.net/web-security/access-control)
  - [OWASP A01:2021 — Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
  - [CWE-284 — Improper Access Control](https://cwe.mitre.org/data/definitions/284.html)

