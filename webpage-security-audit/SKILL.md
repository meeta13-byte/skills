---
name: webpage-security-audit
description: Audit a website or web app codebase (frontend, backend, API) against a 20-point security checklist covering SQL injection, XSS, CSRF, file uploads, IDOR, rate limiting, JWT secrets, exposed API keys, password hashing, MFA, CORS, token storage, client-side-only permissions, row-level security, webhook signatures, SSRF, source maps, default credentials, sensitive logs, and vulnerable dependencies. Use this whenever the user asks to check, audit, scan, harden, or review the security of their website, web app, API, or backend, or asks "is my site secure", "security prompt", "find vulnerabilities", or "pentest checklist", even if they don't mention the number 20.
---

# Web Security Audit (20-point)

Static, read-only security review of the user's OWN website or web app code. Go through all 20 checks. For each one report PASS, FAIL, PARTIAL, or N/A, with evidence.

## Rules
- Only audit code and sites the user owns or has permission to test.
- Read-only by default. Do not change code unless the user asks; propose a fix for every FAIL.
- Do not run exploits, fuzzers, or attacks against live systems. Reading code, configs, and running dependency scanners is fine.
- Redact secrets in the report (show the first 4 characters and the file path only). Never print a full key.
- Cite `file:line` for every finding. If you can't verify something from the code, mark it PARTIAL and say what is needed.

## Step 0: Recon
Identify the stack (framework, language, database, auth provider, hosting), entry points (routes, API handlers, webhooks, upload endpoints), and where config and env files live. Summarize in 5 lines before starting.

## The 20 checks

**Injection and input**
1. **SQL injection**: raw queries built with string concatenation or template strings; ORM `raw`/`execute` calls with user input. Fix: parameterized queries.
2. **XSS**: `dangerouslySetInnerHTML`, `innerHTML`, `v-html`, unescaped template output, missing Content-Security-Policy. Fix: escape or sanitize (DOMPurify), add CSP.
3. **CSRF**: cookie-based sessions on state-changing routes without CSRF tokens or `SameSite` cookies. Fix: CSRF tokens, `SameSite=Lax/Strict`, check Origin.
4. **Insecure file uploads**: no type/size validation, trust of client MIME type, files stored in web root or with user-chosen names, no malware scan. Fix: allow-list types, random names, private storage.

**Access control**
5. **IDOR / broken object-level authorization**: handlers that fetch by id without checking the owner. Fix: always scope queries by the authenticated user.
6. **Missing rate limiting**: login, signup, OTP, password reset, and expensive endpoints with no limits. Fix: per-IP and per-account limits.
7. **Insecure JWT secrets**: short, default, or hard-coded secrets; `alg: none` accepted; no expiry; secret reused across environments. Fix: strong random secret from env, pin the algorithm, short expiry.

**Secrets and credentials**
8. **Exposed API secrets**: keys in source, `.env` committed, secrets in frontend bundles or `NEXT_PUBLIC_`/`VITE_` variables, secrets in git history. Fix: move to server-side env, rotate leaked keys.
9. **Weak password hashing**: MD5, SHA-1/SHA-256 without salt, plain text. Fix: Argon2id, bcrypt, or scrypt.
10. **Missing multi-factor authentication**: no MFA for admin or sensitive accounts. Fix: TOTP or passkeys at least for admins.

**Browser and API surface**
11. **Overly permissive CORS**: `Access-Control-Allow-Origin: *` with credentials, reflected origins. Fix: explicit allow-list.
12. **Auth tokens in local storage**: tokens in `localStorage`/`sessionStorage`. Fix: `HttpOnly`, `Secure`, `SameSite` cookies.
13. **Client-side-only permission enforcement**: role checks only in UI code, hidden buttons treated as security. Fix: enforce on the server for every request.
14. **Missing row-level security**: Supabase/Postgres tables without RLS, or permissive policies (`using (true)`). Fix: enable RLS and write per-user policies.

**Integrations**
15. **Unverified webhook signatures**: Stripe/Razorpay/Twilio/GitHub webhooks processed without signature verification. Fix: verify with the provider's secret and reject replays.
16. **SSRF**: server fetches a user-supplied URL with no allow-list. Fix: allow-list hosts, block private/metadata IP ranges, disable redirects.

**Deployment hygiene**
17. **Exposed source maps**: `.map` files served in production, `devtool` source maps enabled. Fix: disable or upload privately to the error tracker.
18. **Default credentials**: default admin accounts, sample passwords, debug or admin routes left on, default DB passwords. Fix: remove and rotate.
19. **Sensitive data in logs**: passwords, tokens, OTPs, card or personal data in `console.log` or server logs. Fix: redact or remove.
20. **Vulnerable dependencies**: run `npm audit` / `pip-audit` / `yarn audit` or the stack's equivalent, plus a check for outdated frameworks. Fix: upgrade, pin, enable Dependabot.

## Report format
1. Summary: counts of FAIL / PARTIAL / PASS / N/A and the top 3 risks.
2. Table: `# | Check | Status | Severity (Critical/High/Medium/Low) | Evidence (file:line) | Fix`.
3. Fix plan ordered by severity, with the smallest safe change for each.
4. Anything you could not verify and what access you'd need.

Offer to apply the fixes one severity level at a time, and ask before editing.
