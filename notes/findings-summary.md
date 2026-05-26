# Findings Summary

| # | Vulnerability | Severity | CVSS | OWASP | Status |
|---|---------------|----------|------|-------|--------|
| 1 | SQL Injection | Critical | 9.8 | A03:2021 | Confirmed |
| 2 | Reflected XSS | High | 7.4 | A03:2021 | Confirmed |
| 3 | Stored XSS | Critical | 8.2 | A03:2021 | Confirmed |
| 4 | Weak Authentication / No Brute Force Controls | High | 7.5 | A07:2021 | Confirmed |
| 5 | Insecure Session / Cookie Configuration | High | 7.3 | A07:2021 | Confirmed |

## Additional ZAP Findings
| # | Vulnerability | Severity | Status |
|---|---------------|----------|--------|
| 6 | Missing Content Security Policy | Medium | ZAP Confirmed |
| 7 | Missing X-Frame-Options Header | Medium | ZAP Confirmed |
| 8 | Missing CSRF Token | Medium | ZAP Confirmed |
| 9 | Server Version Disclosure | Low | ZAP Confirmed |

## Severity Distribution
- Critical : 2
- High     : 3
- Medium   : 3
- Low      : 1
- Total    : 9
