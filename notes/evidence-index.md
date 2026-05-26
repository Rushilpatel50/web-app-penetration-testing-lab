# Evidence Index

## Lab Setup Evidence
| File | Description |
|------|-------------|
| screenshots/lab-setup/01-vmware-network-editor.png | VMnet1 Host-Only configuration |
| screenshots/lab-setup/02-kali-settings.png | Kali VM hardware settings |
| screenshots/lab-setup/03-metasploitable-settings.png | Metasploitable VM settings |
| screenshots/lab-setup/04-kali-ip-a.png | Kali IP address |
| screenshots/lab-setup/05-metasploitable-ifconfig.png | Target IP address |
| screenshots/lab-setup/06-ping-success.png | Network connectivity confirmed |
| screenshots/lab-setup/07-dvwa-login-page.png | DVWA login page |
| screenshots/lab-setup/08-dvwa-database-setup.png | Database initialized |
| screenshots/lab-setup/09-dvwa-security-low.png | Security level set to Low |

## Reconnaissance Evidence
| File | Description |
|------|-------------|
| screenshots/recon/01-nmap-basic-scan.png | Open ports discovered |
| screenshots/recon/02-nmap-version-scan.png | Service versions identified |
| screenshots/recon/03-nmap-aggressive-scan.png | OS fingerprinting |
| screenshots/recon/04-nmap-web-scan.png | Web service details |

## Finding 1 — SQL Injection Evidence
| File | Description |
|------|-------------|
| screenshots/sqli/01-normal-response.png | Baseline normal behavior |
| screenshots/sqli/02-single-quote-error.png | SQL error confirms injection |
| screenshots/sqli/03-always-true-injection.png | All users returned |
| screenshots/sqli/04-database-name.png | Database name extracted |
| screenshots/sqli/05-table-names.png | Tables enumerated |
| screenshots/sqli/06-column-names.png | Columns enumerated |
| screenshots/sqli/07-credentials-extracted.png | Password hashes dumped |
| screenshots/sqli/08-burp-intercept.png | Request captured in Burp |
| screenshots/sqli/09-burp-repeater.png | Payload replayed in Repeater |
| screenshots/sqli/10-sqlmap-detection.png | SQLMap confirms vulnerability |
| screenshots/sqli/11-sqlmap-databases.png | SQLMap lists databases |
| screenshots/sqli/12-sqlmap-tables.png | SQLMap lists tables |
| screenshots/sqli/13-sqlmap-dump.png | SQLMap dumps full users table |

## Finding 2 — Reflected XSS Evidence
| File | Description |
|------|-------------|
| screenshots/xss-reflected/01-normal-response.png | Normal name input |
| screenshots/xss-reflected/02-alert-popup.png | XSS alert confirmed |
| screenshots/xss-reflected/03-reflected-url.png | Payload in URL bar |
| screenshots/xss-reflected/04-cookie-theft-simulation.png | Cookie in alert |
| screenshots/xss-reflected/05-dom-manipulation.png | Page content injected |
| screenshots/xss-reflected/06-img-tag-xss.png | Filter bypass vector |
| screenshots/xss-reflected/07-burp-intercept.png | Request intercepted |
| screenshots/xss-reflected/08-burp-repeater-response.png | Payload in response |
| screenshots/xss-reflected/09-raw-html-payload.png | Unescaped HTML confirmed |
| screenshots/xss-reflected/10-vulnerable-source-code.png | Source code shows flaw |

## Finding 3 — Stored XSS Evidence
| File | Description |
|------|-------------|
| screenshots/xss-stored/01-normal-entry.png | Normal guestbook entry |
| screenshots/xss-stored/02-payload-submitted.png | XSS payload stored |
| screenshots/xss-stored/03-alert-on-refresh.png | Persistence confirmed |
| screenshots/xss-stored/04-cookie-in-stored-xss.png | Cookie theft simulated |
| screenshots/xss-stored/05-name-field-injection.png | Name field vulnerable |
| screenshots/xss-stored/06-burp-bypass-charlimit.png | Client control bypassed |
| screenshots/xss-stored/07-vulnerable-source.png | Source code analysis |

## Finding 4 — Weak Authentication Evidence
| File | Description |
|------|-------------|
| screenshots/weak-auth/01-normal-login-success.png | Successful login |
| screenshots/weak-auth/02-failed-login-response.png | Failed login message |
| screenshots/weak-auth/03-burp-captured-request.png | GET request with credentials |
| screenshots/weak-auth/04-intruder-positions.png | Attack position configured |
| screenshots/weak-auth/05-intruder-wordlist-loaded.png | Wordlist loaded |
| screenshots/weak-auth/06-intruder-grep-match.png | Success string configured |
| screenshots/weak-auth/07-intruder-attack-results.png | Password identified by length |
| screenshots/weak-auth/08-successful-password-found.png | Welcome message confirmed |
| screenshots/weak-auth/09-hydra-success.png | Hydra CLI confirmation |
| screenshots/weak-auth/10-no-lockout.png | No lockout after failures |
| screenshots/weak-auth/11-credentials-in-url.png | Password exposed in URL |

## Finding 5 — Cookie/Session Evidence
| File | Description |
|------|-------------|
| screenshots/cookies/01-cookies-before-login.png | Pre-login cookie state |
| screenshots/cookies/02-cookies-after-login.png | Same PHPSESSID after login |
| screenshots/cookies/03-javascript-cookie-access.png | JS reads session cookie |
| screenshots/cookies/04-secure-flag-missing.png | No Secure flag shown |
| screenshots/cookies/05-burp-cookie-header.png | Cookie in raw headers |
| screenshots/cookies/06-burp-response-headers.png | Set-Cookie missing flags |
| screenshots/cookies/07-session-id-comparison.png | Session IDs across logins |
| screenshots/cookies/08-cookie-tamper-security-level.png | Security cookie tampered |
| screenshots/cookies/09-session-fixation.png | Same session ID pre/post login |
| screenshots/cookies/10-session-hijack-poc.png | Account accessed without login |

## Automated Scan Evidence
| File | Description |
|------|-------------|
| screenshots/recon/10-zap-alerts-overview.png | All ZAP alerts |
| screenshots/recon/11-zap-sqli-alert-detail.png | SQLi alert detail |
| screenshots/recon/12-zap-cookie-alert-detail.png | Cookie alert detail |
| screenshots/recon/14-zap-report-in-browser.png | HTML report rendered |
| scans/zap-report.html | Full ZAP HTML report |
