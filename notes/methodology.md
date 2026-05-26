## PHASE 3 — RECONNAISSANCE FINDINGS

Date: 21/05/2026
Target IP: 192.168.56.129
Method: Nmap (passive port/service enumeration)

### Open Ports Identified:
| Port  | Service  | Version                  | Notes                    |
|-------|----------|--------------------------|--------------------------|
| 21    | FTP      | vsftpd 2.3.4             | Known backdoor (CVE-2011-2523) |
| 22    | SSH      | OpenSSH 4.7p1            | Outdated version         |
| 23    | Telnet   | Linux telnetd            | Unencrypted — insecure   |
| 80    | HTTP     | Apache 2.2.8             | Hosts DVWA               |
| 3306  | MySQL    | MySQL 5.0.51a            | Database for DVWA        |
| 8180  | HTTP     | Apache Tomcat            | Secondary web service    |

### Web Server Details:
- Server: Apache 2.2.8 (Ubuntu)
- PHP Version: 5.2.4 (outdated, end-of-life)
- Application: DVWA hosted at /dvwa

### OS Detection:
- Linux 2.6.x kernel (identified by Nmap OS scan)

### Recon Tool Used: Nmap 7.x
### Scan Types: Basic, Version/Script, Aggressive, Web-targeted

## FINDING 1 — SQL Injection (Critical)

Severity:       CRITICAL (CVSS 9.8)
Affected URL:   http://192.168.56.129/dvwa/vulnerabilities/sqli/
Parameter:      id (GET)
Tool Used:      Manual testing, Burp Suite Repeater, SQLMap

Description:
The 'id' parameter is passed directly into a MySQL query without
sanitization or parameterization. An attacker can manipulate the
query to extract, modify, or delete database contents.

Proof of Concept:
Payload: 1' UNION SELECT user,password FROM users -- -
Result:  All usernames and MD5 hashed passwords extracted

Impact:
- Full database content extraction (usernames, password hashes)
- Authentication bypass possible
- Potential for data modification or deletion
- In some configurations: OS-level command execution

Evidence:
- screenshots/sqli/03-always-true-injection.png
- screenshots/sqli/07-credentials-extracted.png
- screenshots/sqli/13-sqlmap-dump.png

Remediation:
1. Use Prepared Statements (Parameterized Queries):
   $stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
   $stmt->execute([$id]);

2. Input Validation: Whitelist only numeric input for ID fields
3. Least Privilege: Database user should have SELECT only, not DROP/DELETE
4. Web Application Firewall (WAF): Block common SQLi patterns
5. Error Handling: Never display raw SQL errors to users

OWASP Reference: A03:2021 — Injection


## FINDING 2 — Reflected Cross-Site Scripting (High)

Severity:       HIGH (CVSS 7.4)
Affected URL:   http://192.168.56.129/dvwa/vulnerabilities/xss_r/
Parameter:      name (GET)
Tool Used:      Manual testing, Burp Suite Repeater

Description:
The 'name' parameter is reflected back into the HTML response
without encoding or sanitization. An attacker can inject
JavaScript that executes in the victim's browser when they
visit a crafted URL.

Proof of Concept Payloads:
1. <script>alert('XSS')</script>        → Alert popup confirmed
2. <script>alert(document.cookie)</script> → Session cookie exposed
3. <img src=x onerror=alert('XSS')>    → Filter bypass vector

Attack Scenario:
1. Attacker crafts malicious URL containing XSS payload
2. Attacker sends URL to victim via email/chat/social media
3. Victim clicks link → script executes in victim's browser
4. Attacker steals session cookie → logs in as victim
   without needing password

Impact:
- Session hijacking via cookie theft
- Credential harvesting via fake login forms
- Malware distribution via browser redirects
- Defacement of page content for targeted users

Evidence:
- screenshots/xss-reflected/02-alert-popup.png
- screenshots/xss-reflected/04-cookie-theft-simulation.png
- screenshots/xss-reflected/08-burp-repeater-response.png

Remediation:
1. Output Encoding: Use htmlspecialchars() on all user input
   before rendering in HTML
   $name = htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8');

2. Content Security Policy (CSP): Add response header:
   Content-Security-Policy: script-src 'self'

3. Input Validation: Reject inputs containing HTML special
   characters where not required

4. HTTPOnly Cookie Flag: Prevents JavaScript from accessing
   session cookies even if XSS occurs

OWASP Reference: A03:2021 — Injection (XSS subtype)

## FINDING 3 — Stored Cross-Site Scripting (Critical)

Severity:       CRITICAL (CVSS 8.2)
Affected URL:   http://192.168.56.129/dvwa/vulnerabilities/xss_s/
Parameters:     txtName (POST), mtxMessage (POST)
Tool Used:      Manual testing, Burp Suite Repeater

Description:
Both the Name and Message fields of the guestbook form accept
and store unsanitized user input in the MySQL database. The
stored content is retrieved and rendered without output encoding,
causing injected scripts to execute in every visitor's browser.

Proof of Concept:
Payload: <script>alert(document.cookie)</script>
Submitted via: Message field (POST)
Result: Alert fires on every page load for all users
        Session cookie exposed to JavaScript

Character Limit Bypass:
The HTML maxlength attribute on the Name field was bypassed
using Burp Suite Repeater to send a longer payload directly
via HTTP POST, confirming client-side controls are not a
sufficient security measure.

Attack Scenario:
1. Attacker submits XSS payload in guestbook
2. Payload saved permanently in database
3. Every user who visits the page gets attacked silently
4. Attacker collects session cookies from all visitors
5. Attacker hijacks accounts without needing passwords

Impact:
- Mass session hijacking of all site visitors
- Persistent malware/phishing delivery
- No victim interaction required beyond page visit
- Client-side defenses (maxlength) bypassable via proxy

Evidence:
- screenshots/xss-stored/02-payload-submitted.png
- screenshots/xss-stored/03-alert-on-refresh.png
- screenshots/xss-stored/04-cookie-in-stored-xss.png
- screenshots/xss-stored/05-burp-bypass-charlimit.png

Remediation:
1. Output Encoding: htmlspecialchars() on all stored content
   before rendering in HTML

2. Input Validation: Server-side length limits and character
   whitelisting — never rely on HTML maxlength alone

3. Content Security Policy header:
   Content-Security-Policy: script-src 'self'

4. Prepared Statements: Already needed for SQLi — also
   prevents second-order injection from stored payloads

5. HTTPOnly Cookie Flag: Blocks JavaScript cookie access
   even if XSS executes successfully

6. Regular Output Review: Audit all database-sourced content
   rendered in HTML responses

OWASP Reference: A03:2021 — Injection (XSS subtype)
Difference from Finding 2: Stored/Persistent vs Reflected

## FINDING 4 — Weak Authentication & Missing Brute Force Controls (High)

Severity:       HIGH (CVSS 7.5)
Affected URL:   http://192.168.56.129/dvwa/vulnerabilities/brute/
Parameters:     username, password (GET)
Tools Used:     Burp Suite Intruder, Hydra

Description:
The login form lacks essential brute force protections. Combined
with weak default credentials and GET-based credential submission,
the application is trivially susceptible to automated password
attacks.

Vulnerabilities Identified:
1. No account lockout after repeated failures
2. No CAPTCHA or bot detection
3. No rate limiting between attempts
4. Credentials transmitted via GET (visible in logs/history)
5. Weak default credentials (admin/password)

Proof of Concept:
Tool: Burp Suite Intruder (Sniper attack)
Wordlist: 10-password custom list
Result: Correct password 'password' found in attempt #9
Time: Under 3 seconds for 10 attempts

Tool: Hydra
Command: hydra -l admin -P wordlist.txt [target] http-get-form
Result: [80][http-get-form] login: admin   password: password

Impact:
- Automated tools can test millions of passwords undetected
- Weak default credentials trivially guessed
- Credentials exposed in server logs via GET method
- No mechanism to detect or alert on brute force activity
- Full account takeover achievable without technical skill

Evidence:
- screenshots/weak-auth/03-burp-captured-request.png
- screenshots/weak-auth/07-intruder-attack-results.png
- screenshots/weak-auth/08-successful-password-found.png
- screenshots/weak-auth/09-hydra-success.png

Remediation:
1. Account Lockout: Lock account for 15–30 minutes after
   5 consecutive failed attempts

2. Rate Limiting: Enforce minimum 1–2 second delay between
   login attempts per IP

3. CAPTCHA: Implement after 3 failed attempts to block
   automated tools

4. POST + HTTPS: Move login form to POST method over HTTPS
   to prevent credentials appearing in logs/history

5. Strong Password Policy: Minimum 12 characters,
   complexity requirements, no default credentials

6. Multi-Factor Authentication (MFA): Most effective defense
   — compromised password alone is insufficient

7. Monitoring & Alerting: Log and alert on repeated
   authentication failures from same IP

OWASP Reference: A07:2021 — Identification and Authentication Failures

## FINDING 5 — Insecure Session and Cookie Configuration (High)

Severity:       HIGH (CVSS 7.3)
Affected URL:   http://192.168.56.129/dvwa/ (all pages)
Component:      PHPSESSID session cookie, security cookie
Tools Used:     Browser DevTools, Burp Suite Proxy

Description:
The application issues session cookies without critical security
attributes. Combined with missing session regeneration after
login, this exposes all authenticated sessions to theft,
fixation, and hijacking attacks.

Vulnerabilities Found:

[A] Missing HttpOnly Flag
    Cookie readable via JavaScript (document.cookie)
    Directly exploitable via XSS findings (Finding 2 & 3)
    Proof: document.cookie returned PHPSESSID in browser console

[B] Missing Secure Flag
    Cookie transmitted over plain HTTP — visible on network
    Any network observer can capture session token

[C] Missing SameSite Attribute
    Cross-site requests carry session cookie automatically
    Enables Cross-Site Request Forgery (CSRF) attacks

[D] No Session Regeneration After Login
    PHPSESSID unchanged before and after authentication
    Enables session fixation attacks:
    - Attacker knows pre-auth session ID
    - Victim logs in — same ID still valid
    - Attacker uses pre-auth ID to access authenticated session

[E] Security Level Controlled by Client Cookie
    security=low cookie modifiable by any user via proxy
    Application security should never be client-controlled

[F] Session Hijacking Demonstrated
    Copied victim PHPSESSID into attacker browser
    Gained full authenticated access without credentials

Proof of Concept:
1. document.cookie → returned full session token via JS
2. Burp captured cookie in plaintext HTTP headers
3. Modified security=low to security=impossible via Burp
4. Copied PHPSESSID to private window → authenticated as admin

Impact:
- XSS + missing HttpOnly = automatic session theft at scale
- Missing Secure flag = session exposed on any HTTP network
- Session fixation = account takeover without password
- Client-controlled security = all protections bypassable
- No expiry = stolen sessions valid indefinitely

Evidence:
- screenshots/cookies/03-javascript-cookie-access.png
- screenshots/cookies/06-burp-response-headers.png
- screenshots/cookies/08-cookie-tamper-security-level.png
- screenshots/cookies/09-session-fixation.png
- screenshots/cookies/10-session-hijack-poc.png

Remediation:
1. Set HttpOnly flag on all session cookies:
   session_set_cookie_params(['httponly' => true]);

2. Set Secure flag (requires HTTPS deployment):
   session_set_cookie_params(['secure' => true]);

3. Set SameSite attribute:
   session_set_cookie_params(['samesite' => 'Strict']);

4. Regenerate Session ID after login:
   session_regenerate_id(true);

5. Move security controls entirely server-side —
   never trust client-supplied values for access control

6. Set short session timeout:
   ini_set('session.gc_maxlifetime', 1800); // 30 min

7. Combined PHP secure session setup:
   session_set_cookie_params([
     'lifetime' => 1800,
     'secure'   => true,
     'httponly' => true,
     'samesite' => 'Strict'
   ]);
   session_start();
   session_regenerate_id(true);

OWASP Reference: A07:2021 — Identification and Authentication Failures

## PHASE 9 — ZAP SCAN vs MANUAL TESTING COMPARISON

| Vulnerability          | Manual Found | ZAP Found | Notes                        |
|------------------------|--------------|-----------|------------------------------|
| SQL Injection          | ✅ Yes       | ✅ Yes    | ZAP confirmed, less detail   |
| Reflected XSS          | ✅ Yes       | ✅ Yes    | ZAP confirmed                |
| Stored XSS             | ✅ Yes       | ⚠️ Maybe   | ZAP may miss stored payloads |
| Weak Authentication    | ✅ Yes       | ✅ Yes    | ZAP flagged no CSRF token    |
| Missing HttpOnly       | ✅ Yes       | ✅ Yes    | ZAP confirmed                |
| Missing Secure Flag    | ✅ Yes       | ✅ Yes    | ZAP confirmed                |
| Missing CSP Header     | ❌ Not tested| ✅ Yes    | ZAP found additional finding |
| X-Frame-Options Missing| ❌ Not tested| ✅ Yes    | ZAP found additional finding |
| Server Version Exposed | ❌ Not tested| ✅ Yes    | Apache/2.2.8 in headers      |

Key Observations:
- ZAP confirmed all 5 manual findings
- ZAP found 3 additional informational/medium findings
- ZAP missed nuance: severity context, chained attack scenarios
- Manual testing found exploitation depth ZAP cannot:
  * Full credential dump via SQLMap
  * Session hijacking proof of concept
  * Character limit bypass via Burp
  * Cookie tampering impact demonstration
- Conclusion: Automated + Manual = most complete coverage

## ZAP SCAN SUMMARY

Scan Date:    25/05/2026
Target:       http://192.168.56.129/dvwa
Scanner:      OWASP ZAP 2.x
Scan Type:    Spider + Active Scan (authenticated)
Duration:     ~10 minutes

Alert Summary:
  High Risk:    2  (SQL Injection, XSS)
  Medium Risk:  3  (CSP missing, X-Frame-Options, CSRF)
  Low Risk:     4  (Cookie flags, Server version disclosure)
  Information:  3  (Session handling, comments)
  Total:        12 alerts

Notable Automated Findings:
1. SQL Injection confirmed on 'id' parameter
2. Reflected XSS confirmed on 'name' parameter  
3. CSP header absent across all pages
4. X-Frame-Options missing — clickjacking possible
5. CSRF tokens absent on authentication forms
6. Server discloses Apache version in headers
7. Both session cookies missing HttpOnly + Secure flags

Automated vs Manual Assessment:
- Automation is faster but shallower
- Manual testing revealed chained attack impact
- Combined approach provides most complete risk picture
- ZAP report serves as independent corroboration of findings
