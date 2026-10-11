---
name: apk-security-audit
description: Audit an Android app (source project or built APK) against a 20-point security checklist adapted for mobile: SQL injection, WebView XSS, deep-link/intent forgery, insecure uploads, IDOR, rate limiting, JWT secrets, hard-coded API keys, weak PIN/password hashing, missing MFA/biometrics, network security config, token storage, client-side-only permissions, Firebase/Firestore rules, webhook signatures, SSRF, release build hardening, default credentials, sensitive logs, and vulnerable dependencies. Use this whenever the user asks to check, audit, scan, harden, or review the security of their Android app, APK, Kotlin/Java project, or its Firebase/backend, or asks "is my app secure", "security prompt", or "find vulnerabilities in my APK", even if they don't mention the number 20.
---

# Android APK Security Audit (20-point)

Static, read-only security review of the user's OWN Android app and its backend (Firebase rules, Cloud Functions, APIs). Go through all 20 checks. For each report PASS, FAIL, PARTIAL, or N/A, with evidence.

## Rules
- Only audit apps and backends the user owns or has permission to test.
- Read-only by default. Do not change code unless the user asks; propose a fix for every FAIL.
- Do not attack live servers or run exploits. Reading the project, `AndroidManifest.xml`, Gradle files, rules files, and running dependency scanners is fine. Decompiling the user's own APK to check what ships is fine.
- Redact secrets in the report (first 4 characters and the file path only).
- Cite `file:line` for every finding. If it can't be verified from the code, mark PARTIAL and say what is needed.

## Step 0: Recon
Identify: package name, `minSdk`/`targetSdk`, language (Kotlin/Java), UI (Compose/XML), auth (Firebase Auth, custom), data layer (Room/SQLite, Firestore, REST), backend (Cloud Functions, own server), and the release build config. Read `AndroidManifest.xml`, `app/build.gradle(.kts)`, `network_security_config.xml`, and Firebase rules files. Summarize in 5 lines.

## The 20 checks (mobile version)

**Injection and input**
1. **SQL injection**: `rawQuery`/`execSQL` or Room `@RawQuery` built by string concatenation; unprotected `ContentProvider`s; backend queries built from app input. Fix: parameterized queries, `selectionArgs`, protect providers.
2. **XSS (WebView)**: `setJavaScriptEnabled(true)` with untrusted content, `addJavascriptInterface`, `loadUrl` with user-controlled URLs, file access enabled. Fix: disable JS unless needed, allow-list URLs, remove risky interfaces.
3. **CSRF / intent forgery**: exported activities, services, receivers, or deep links that perform actions without confirmation or authentication; implicit intents carrying sensitive data; mutable `PendingIntent`. Fix: `android:exported="false"`, validate callers, `FLAG_IMMUTABLE`.
4. **Insecure file handling and uploads**: no type/size validation before upload, files saved to shared external storage, `file://` URIs, path traversal in downloads. Fix: validate, use `FileProvider`, app-private storage.

**Access control**
5. **IDOR / broken object-level authorization**: API calls or database paths keyed by user-controlled ids (phone number, email, sequential id); the server not checking ownership. Fix: key by `FirebaseAuth` uid and enforce ownership on the server and in rules.
6. **Missing rate limiting**: OTP send/verify, login, and PIN attempts unlimited; no Firebase App Check. Fix: server-side limits, lockout after failed PIN attempts, enable App Check.
7. **Insecure JWT / token handling**: tokens decoded and trusted on-device, long-lived tokens, backend not verifying ID tokens (`verifyIdToken`), hard-coded signing secrets. Fix: verify tokens on the server, short expiry.

**Secrets and credentials**
8. **Exposed API secrets**: keys, Twilio/payment secrets, or service-account JSON in source, `BuildConfig`, `strings.xml`, `gradle.properties`, or committed `.env`; secrets recoverable from the APK. Fix: move secrets to the backend, rotate leaked keys. (Firebase `google-services.json` API keys are identifiers, not secrets, but must be restricted by package + SHA and protected by rules.)
9. **Weak password/PIN hashing**: PIN or password stored plain, or hashed with MD5/SHA without salt. Fix: PBKDF2/Argon2/bcrypt with a per-user salt, keys in the Android Keystore, attempt limits.
10. **Missing MFA / biometrics**: no step-up for sensitive actions (payments, PIN reset, export). Fix: BiometricPrompt or re-authentication for sensitive operations; phone OTP plus PIN counts as two factors only if enforced server-side.

**Device and network surface**
11. **Network security (the mobile "CORS" check)**: `usesCleartextTraffic="true"`, permissive `network_security_config`, trust of user CAs, disabled certificate validation (`TrustManager` that trusts all), no pinning for critical APIs. Also check the backend's CORS if it serves a web client. Fix: HTTPS only, remove custom trust-all code, consider pinning.
12. **Auth tokens / data in insecure storage**: tokens, PINs, or personal data in plain `SharedPreferences`, SQLite, or files; `android:allowBackup="true"` exposing data. Fix: EncryptedSharedPreferences or Keystore, `allowBackup="false"` or backup rules.
13. **Client-side-only permission enforcement**: role/premium/limit checks only in the app UI or a local flag. Fix: enforce on the server and in database rules.
14. **Missing row-level security**: Firestore, Realtime Database, or Storage rules like `allow read, write: if true` or `if request.auth != null` without ownership checks; Supabase tables without RLS. Fix: per-user rules (`request.auth.uid == userId`) and field validation.

**Integrations**
15. **Unverified webhook signatures**: Cloud Functions or server endpoints for payment or SMS webhooks that skip signature verification. Fix: verify the provider signature and reject replays.
16. **SSRF**: backend functions that fetch URLs supplied by the app without an allow-list; in-app URL loading from untrusted input. Fix: allow-list hosts, block private/metadata IP ranges.

**Release and hygiene**
17. **Release build hardening (the mobile "source map" check)**: `isMinifyEnabled=false`, no R8/ProGuard, `debuggable=true`, mapping files or debug symbols shipped, debug keystore used for release, `android:testOnly`. Fix: enable minify and shrink, release signing config, never ship debug builds.
18. **Default credentials**: default or test PINs, hard-coded test accounts or Firebase test phone numbers active in production, debug endpoints or feature flags left on. Fix: remove, rotate.
19. **Sensitive data in logs**: `Log.d/i/v`, `println`, or crash reports containing OTPs, tokens, PINs, phone numbers, or financial data. Fix: remove or strip logs in release (R8 `-assumenosideeffects`), redact crash reports.
20. **Vulnerable dependencies**: run `./gradlew dependencies` and a scanner such as OWASP dependency-check or `osv-scanner` if available; check for outdated Firebase BoM, AndroidX, OkHttp, and WebView-related libraries; check `targetSdk` is current. Fix: upgrade, pin versions, enable Dependabot.

## Optional: built APK check
If an APK is provided, list its contents and check that no secrets, debug files, or unobfuscated sensitive strings are present (`apktool` or `unzip -l`, then `grep` for key patterns). Treat this as inspecting the user's own build only.

## Report format
1. Summary: counts of FAIL / PARTIAL / PASS / N/A and the top 3 risks.
2. Table: `# | Check | Status | Severity (Critical/High/Medium/Low) | Evidence (file:line) | Fix`.
3. Fix plan ordered by severity, with the smallest safe change for each.
4. Anything you could not verify (for example Firebase rules deployed in the console) and what access you'd need.

Offer to apply fixes one severity level at a time, and ask before editing.
