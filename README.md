# ordersmart-security-audit
Security vulnerability assessment of a live AI web application built with Claude Code.
# OrderSmarter Security Audit Report

**Application:** OrderSmarter (useordersmart.com)  
**Type:** AI-powered web application for restaurant owners  
**Stack:** Node.js, PostgreSQL, Railway, Claude API  
**Audit Date:** March 2026  
**Auditor:** Joshua Leman  

---

## Executive Summary

OrderSmarter is a live AI web application used by real restaurant owners. 
This report documents 7 security vulnerabilities discovered and remediated 
during a self-directed security audit conducted over 25 days of cybersecurity training.

All vulnerabilities have been fixed. No data breach occurred.

---

## Findings Overview

| # | Finding | Severity | Status |
|---|---------|----------|--------|
| 1 | File Permission Misconfiguration | High | Fixed |
| 2 | Broken Access Control | Critical | Fixed |
| 3 | Weak Password Hashing | High | Fixed |
| 4 | No Account Lockout | Medium | Fixed |
| 5 | Vulnerable Dependencies | High | Fixed |
| 6 | Zero Security Logging | Medium | Fixed |
| 7 | Unauthenticated AI Calls | High | Fixed |

---

## Finding 1 — File Permission Misconfiguration

**Severity:** High  
**OWASP Category:** Security Misconfiguration (#5)

**Description:**  
My .env file containing API keys, database credentials, and JWT secrets 
was configured with chmod 644 permissions, making it readable by all users on the server.

**Impact:**  
Any of my users2 with server access could read all secret keys including the Anthropic API key, 
database password, and JWT signing secret.

**Fix:**  
Changed file permissions to chmod 600, restricting read/write access to owner only.

---

## Finding 2 — Broken Access Control

**Severity:** Critical  
**OWASP Category:** Broken Access Control (#1)

**Description:**  
11 API routes had no authorization checks. Any authenticated user could read, 
modify, or delete any other restaurant's data.

**Affected Routes:**  
- GET /history/:restaurantId — anyone could read chat history  
- DELETE /history/:restaurantId — anyone could delete chat history  
- GET /supplier-emails/:restaurantId — exposed supplier emails  
- POST /send — anyone could trigger supplier emails  
- Square.js sync, status, disconnect endpoints  

**Impact:**  
Complete exposure of all restaurant owner data including AI chat history, 
supplier relationships, and Square payment integrations.

**Fix:**  
Applied verifyRestaurantOwner() middleware to all affected routes.

---

## Finding 3 — Weak Password Hashing

**Severity:** High  
**OWASP Category:** Cryptographic Failures (#2)

**Description:**  
Passwords were hashed using SHA-256 with a static salt stored in the .env file. 
SHA-256 is a general purpose hashing algorithm not designed for password storage.

**Impact:**  
An attacker who obtained the database could crack passwords at billions of 
attempts per second using GPU acceleration.

**Fix:**  
Moved to bcrypt with 12 rounds. Implemented silent migration for existing 
users — legacy SHA-256 hashes are upgraded to bcrypt on next login. 
No password reset required.

---

## Finding 4 — No Account Lockout

**Severity:** Medium  
**OWASP Category:** Authentication Failures (#7)

**Description:**  
No limit existed on failed login attempts, allowing unlimited brute force attacks 
against any account.

**Impact:**  
Attacker could systematically try all common passwords against any account 
without being blocked.

**Fix:**  
Added account lockout after 5 failed attempts with 15 minute cooldown. 
Lockout resets automatically on successful login.

---

## Finding 5 — Vulnerable Dependencies

**Severity:** High  
**OWASP Category:** Vulnerable and Outdated Components (#6)

**Description:**  
npm audit identified 3 high severity vulnerabilities in production dependencies.

**Affected Packages:**  
- express-rate-limit — IPv4-mapped IPv6 addresses could bypass rate limiting  
- nodemailer — potential DoS vulnerability  
- effect/prisma — AsyncLocalStorage context issue  

**Impact:**  
The express-rate-limit vulnerability directly undermined the rate limiting 
implemented in Finding 4, allowing bypass via VPN or IPv6 addresses.

**Fix:**  
Updated express-rate-limit and nodemailer. Reverted prisma to clean version 6.0.0.

---

## Finding 6 — Zero Security Logging

**Severity:** Medium  
**OWASP Category:** Security Logging and Monitoring Failures (#9)

**Description:**  
No security events were being logged. Failed logins, authorization failures, 
rate limit hits, and suspicious activity were completely invisible.

**Impact:**  
Active attacks would go completely undetected. No ability to investigate 
incidents or identify patterns of malicious behavior.

**Fix:**  
Implemented structured security logging covering failed logins with IP and email, 
account lockouts, 403 authorization failures, successful logins, and password resets. 
All events visible in real time via Railway Deploy Logs.

---

## Finding 7 — Unauthenticated AI Calls

**Severity:** High  
**OWASP Category:** LLM Model Denial of Service

**Description:**  
The requireSubscription middleware contained a critical flaw — missing token 
requests were passed through to route handlers which never checked authentication 
before making Claude API calls. The upload endpoint had no authentication at all.

**Evidence:**  
An account using support@microsoft.com was found making unauthenticated 
AI calls, consuming API credits without an account.

**Impact:**  
Any unauthenticated user could consume Anthropic API credits. 
With auto-reload enabled this could result in unlimited charges.

**Fix:**  
Fixed requireSubscription to return 401 immediately on missing or expired tokens. 
Added authentication check to upload endpoint before any file processing. 
Added per-user rate limiting of 20 AI messages per hour stacked on existing IP limits.

---

## Recommendations

1. Add email verification for new signups
2. Implement CSRF tokens on all state-changing requests
3. Add Content Security Policy headers to prevent XSS
4. Remove Server header to prevent infrastructure disclosure
5. Regular npm audit as part of deployment pipeline
6. Add spending alerts on Anthropic account

---

## Conclusion

This audit identified and remediated 7 security vulnerabilities across 
authentication, authorization, cryptography, dependency management, 
logging, and AI-specific attack surfaces.

All findings were discovered through manual code review and security 
testing as part of my structured cybersecurity learning curriculum. 
No automated scanning tools were used.

The most critical finding — Broken Access Control — had real world 
impact potential as the application serves live restaurant owners with 
sensitive business data.

---

*Audit conducted as part of my AI security specialization curriculum. 
All vulnerabilities were responsibly disclosed and fixed by the application owner.*
