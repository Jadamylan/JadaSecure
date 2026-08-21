# JadaSecure

Personal security standard for every app I build, starting with Scentiendo.

This file is the source of truth. It combines the strongest **attack checks** from [vibe-check](https://github.com/benavlabs/vibe-check) with the strongest **ops / launch checks** from [vibe-security](https://github.com/astoj/vibe-security), plus rules learned from shipping a Supabase + TanStack Start app.

It will not make an app bulletproof. It covers the mistakes that actually take vibe-coded apps down. When there is real traction and real user data, hire a pentester.

---

## How to use this

Three layers. Do not skip layer 3.

1. **While coding** — Put this file in the project (`Jadasecure.md`). Tell Cursor: *Follow JadaSecure. Do not ship a change that violates Hard Rules.*
2. **Before launch** — Tell Cursor: *Audit this repo against Jadasecure.md. Go through Ship blockers, then every Launch checklist item. Write findings. Fix blockers before anything else.*
3. **Before real users** — You run **Manual tests** yourself. If you only have time for five, do Ship blockers.

Copy this file into every new repo. Do not rely on memory.

---

## Ship blockers

These five took down real products. An app does not ship if any fail.

| # | Check | Pass |
|---|--------|------|
| 1 | Database is not publicly readable | Anon/publishable key cannot read other users' rows |
| 2 | Protected APIs reject strangers | No session → 401. Non-admin → 403 on admin routes |
| 3 | No secrets in git | `.env` untracked. No live keys in source or history |
| 4 | Cannot open someone else's data by ID | User A cannot GET/PUT User B's resource |
| 5 | No secret keys in the browser | DevTools Sources / Network show only publishable keys |

---

## Hard Rules

Non-negotiable for generated code. Agents must follow these without being asked.

### Secrets

- Never put API keys, database credentials, tokens, or service-role keys in frontend code (`src/`, `app/`, `pages/`, `components/`, `public/`).
- Never put secrets in `VITE_`, `NEXT_PUBLIC_`, or `REACT_APP_` variables. Those are baked into the client bundle.
- Publishable keys are allowed in the client (Supabase publishable/anon). Service role, Gemini, Stripe secret, and webhook secrets are not.
- Never hardcode credentials. Server-only env vars, loaded only in server modules.
- `.env` must be in `.gitignore` before the first commit. `.env.example` has placeholders only.
- After a leak: rotate the key. `.gitignore` does not un-commit history.

### Database

- Use a maintained platform or ORM (Supabase, Prisma, etc.). No string-built SQL.
- Enable Row Level Security (or equivalent) on every table before launch. Default deny. Policies scoped to `auth.uid()` / `request.auth.uid`.
- Never `USING (true)` or unrestricted `FOR ALL` on **user-owned** data.
- Public catalogs (read-only product data) may be `SELECT` public. Writes must never be public. Document the exception in the repo.
- Firebase rules must require auth and scope to `request.auth.uid`.
- Never deserialize untrusted data (`pickle`, etc.). JSON only.

### Auth and access

- Use a maintained auth library (Supabase Auth, Clerk, Auth0, NextAuth). Do not invent password storage.
- If hashing passwords yourself: bcrypt, Argon2, or scrypt only. Never MD5, SHA-1, or plain SHA-256.
- Enable MFA when the product holds anything you would mind leaking.
- Auth middleware runs **before** the handler, not inside it.
- Unauthenticated → **401**. Authenticated but not owner / not admin → **403**.
- Being logged in is not ownership. Every ID in a URL, query, or body needs `current_user.id == resource.owner_id` (and RLS that matches).
- Session cookies: `httpOnly`, `secure`, `SameSite=Lax` (or Strict). Bearer + localStorage still needs CSRF protection on cookie-based mutations.
- Expensive endpoints (AI, scraping, service-role, bulk jobs) require auth **or** a documented guest exception plus a tight rate limit.

### Input, XSS, files

- Validate all input on the server. Client checks are UX only.
- Parameterized queries / ORM only.
- Never `dangerouslySetInnerHTML`, `v-html`, or `innerHTML` with user content unless sanitized (DOMPurify).
- Uploads: detect type from **magic bytes**, not the filename. Cap size server-side. Rename (UUID or `{user_id}/…`). Store on a bucket/CDN, not executed on the app origin.

### SSRF

If the app fetches a URL the user provided (previews, proxies, “import from URL”):

- Allow `http` and `https` only.
- Resolve DNS and block private IPs before requesting: `127.0.0.0/8`, `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `169.254.0.0/16`, `::1`.
- Fixed vendor URLs (Gemini, Supabase, your own API) are fine. Never pass user strings into `fetch`.

### Headers, CORS, HTTPS

On every response:

- `Content-Security-Policy` (start from `default-src 'self'`, then add only what the app needs)
- `Strict-Transport-Security: max-age=31536000; includeSubDomains`
- `X-Frame-Options: DENY`
- `X-Content-Type-Options: nosniff`
- `Referrer-Policy: strict-origin-when-cross-origin`

Also:

- Production is HTTPS only. Certificates auto-renew.
- CORS origin is an explicit allowlist of real domains. Never `*` with `credentials: true`. Prefer never `*` at all.

### Rate limits and abuse

- Login, signup, and password reset are rate-limited (provider limits plus app limits on anything you own).
- AI, scan, enrich, search, and other costly routes are rate-limited.
- Do not trust `X-Forwarded-For` unless you are behind a proxy you control (`cf-connecting-ip` is preferred).
- CAPTCHA when bots are actually hitting auth.

### Payments

When Stripe (or similar) exists:

- Verify webhook signatures on every request. Reject missing/invalid signatures.
- Store processed event IDs; skip duplicates.
- Handle success **and** failure/cancel/past_due, not only `payment_intent.succeeded`.

### Errors

- Clients get generic copy: “Something went wrong” / “Invalid credentials”.
- Stack traces, SQL, paths, and library names stay in server logs.
- Debug / pretty error pages off in production.

### Dependencies

- Before installing a package an AI suggested: it exists on the official registry, has real download history, and is not a lookalike name.
- Commit the lockfile (`bun.lock`, `package-lock.json`, etc.).
- Keep dependencies updated. Turn on Dependabot or equivalent.

---

## Launch checklist

Walk this before users. Mark N/A only when the feature does not exist (and say why).

### A. Identity

- [ ] Trusted auth library, not a custom password table
- [ ] Password reset is time-limited; sessions expire
- [ ] MFA available or scheduled
- [ ] Every user-data API authenticates **before** work starts
- [ ] Roles exist if there are admins; least privilege

### B. Data

- [ ] RLS (or equivalent) on every table
- [ ] User tables: no `USING (true)`
- [ ] Service-role / admin DB keys only on the server
- [ ] Anon key cannot dump `users`, inventory, messages, or profiles that should be private

### C. Secrets and git

- [ ] `.env` gitignored and untracked
- [ ] No secrets in git history (`gitleaks` or equivalent)
- [ ] `.env.example` is placeholders
- [ ] No `VITE_` / `NEXT_PUBLIC_` secret keys

### D. App surface

- [ ] IDOR: ownership checked on GET and writes
- [ ] CSRF: SameSite cookies or CSRF tokens on state-changing routes
- [ ] XSS: user text rendered as text
- [ ] SQL: no concatenated queries
- [ ] SSRF: no user URL fetch, or IP allowlist in place
- [ ] Uploads: magic bytes, size cap, rename, bucket
- [ ] Guest-expensive routes: rate-limited
- [ ] Headers present (see Hard Rules)
- [ ] CORS allowlist
- [ ] Payments: signed, idempotent webhooks (or N/A)

### E. Run the product

- [ ] HTTPS in production
- [ ] Managed host (Vercel, Lovable, Fly, AWS, GCP, etc.) with automatic patches
- [ ] Backups on and restore tested (Supabase PITR / host backups)
- [ ] Errors logged server-side; something alerts you (even email)
- [ ] Privacy: you can explain what you collect; users can get or delete their data if the law that applies to you requires it
- [ ] You know how to rotate every key (Supabase, Gemini, Stripe) in under 30 minutes
- [ ] If you use Terraform/CloudFormation: scan it (Checkov or equivalent); least-privilege cloud roles

---

## Manual tests

Run these on **your** staging or production app, with **your** keys. Replace URLs and names.

### 1. Database not publicly queryable

```bash
curl "https://YOUR_PROJECT.supabase.co/rest/v1/user_inventory?select=*" \
  -H "apikey: YOUR_PUBLISHABLE_KEY"
```

Pass: empty array or permission error. Fail: other people's rows.

Repeat for every sensitive table (`profiles`, messages, orders, …).

### 2. APIs reject logged-out users

Sign in, copy a protected request from DevTools. Sign out. Replay it with no cookie and no `Authorization`.

Pass: 401. Fail: data.

Call an admin route as a normal user. Pass: 403.

### 3. No secrets in git

```bash
git ls-files .env
grep "\.env" .gitignore
```

Pass: first command prints nothing; `.env` is ignored.

Then search history (install [gitleaks](https://github.com/gitleaks/gitleaks) if needed):

```bash
gitleaks detect --source . --verbose
```

Pass: no leaks.

### 4. IDOR

Two accounts. As A, request B's resource by ID (GET and PUT/PATCH/DELETE).

Pass: 403 or empty. Fail: B's data or a successful write.

### 5. No secrets in the browser

DevTools → Sources: search `sk_`, `sb_secret`, `GEMINI`, `SERVICE_ROLE`, `AKIA`, `Bearer`.  
Network: no secret keys going to third parties from the client.

Pass: only publishable/anon keys.

### 6. SSRF (only if you fetch user URLs)

Submit `http://127.0.0.1/`, `http://localhost/`, `http://169.254.169.254/`. Pass: rejected before a request.

### 7. CSRF

Logged-in browser. Cross-origin auto-submitting form to a state-changing URL. Pass: no change.

### 8. Headers

```bash
curl -I https://your-app.com
```

Pass: CSP, HSTS, `X-Frame-Options`, `X-Content-Type-Options` present.  
Or [securityheaders.com](https://securityheaders.com).

### 9. CORS

```bash
curl -I -H "Origin: https://evil.example" https://your-app.com/api/anything
```

Pass: no `*`, and not echoing `evil.example`.

### 10. Login brute force

Many failed logins from one place. Pass: 429 (or the auth provider blocks). Fail: unlimited 401s.

### 11. Injection / XSS in inputs

Search, login, bios: `' OR '1'='1` and `<script>alert(1)</script>`.

Pass: no extra data, no SQL errors, no alert — shown as text.

### 12. Webhooks (if you take payments)

POST a fake event with no valid signature. Pass: 400, not processed. Same valid event twice: processed once.

### 13. Uploads

File named `photo.jpg` whose contents are HTML. Pass: rejected, or downloaded, or other domain — never executed on your origin.

### 14. Errors

Send garbage JSON and a lone `'`. Pass: generic message. Fail: stack, SQL, file paths.

### 15. Dependencies

`bun audit` / `npm audit`. New packages: real npm page, real downloads, not published yesterday under a typo name.

---

## Agent prompt

Paste this when you want an audit:

```
Audit this repository against Jadasecure.md.

1. Check Ship blockers first. If any fail, treat them as launch blockers.
2. Then walk the Launch checklist. For each item: PASS, FAIL, or N/A (with a one-line reason).
3. Do not invent exploits. Describe what is wrong in the code and how to fix it.
4. Implement Hard Rule violations I confirm. Leave product decisions (MFA, CAPTCHA) listed, not silently skipped.
5. Write a short summary: blockers, then everything else.
```

---

## Sources

- [benavlabs/vibe-check](https://github.com/benavlabs/vibe-check) — attack list, agent rules, manual tests
- [astoj/vibe-security](https://github.com/astoj/vibe-security) — auth/MFA, hosting, backups, privacy, monitoring, incident response
- Scentiendo (`scentlayer-studio`) — guest AI routes must be rate-limited; publishable `VITE_` keys are OK; service role and model keys are not
