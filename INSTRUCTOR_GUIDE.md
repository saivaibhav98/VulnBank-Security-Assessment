# VulnBank — Instructor Guide / Answer Key

**Do not distribute this file to students before grading.** Share the
`README.md` and app files only.

## Vulnerability inventory (12 findings)

| # | Vulnerability | Endpoint | Severity | Proof |
|---|---|---|---|---|
| 1 | SQL Injection — auth bypass | `POST /login` | Critical | Login as `admin' -- ` with any password |
| 2 | Weak/unsalted password hashing (MD5) | storage layer | Medium | Extract hashes via #11, crack with hashcat/john |
| 3 | No account lockout / brute-force protection | `POST /login` | Medium | Hydra/Burp Intruder against `/login` succeeds unthrottled |
| 4 | Reflected XSS | `GET /search?q=` | High | `q=<script>alert(document.cookie)</script>` |
| 5 | Stored XSS | `POST /profile/<id>/edit` (bio field) | High | Save `<img src=x onerror=alert(1)>` as bio, view profile |
| 6 | IDOR — view any user's profile | `GET /profile/<id>` | High | Logged in as alice (id=1), visit `/profile/4` (admin) |
| 7 | Broken access control — role trusted from client cookie | `GET /admin` | Critical | Set cookie `role=admin` manually, visit `/admin` as any user |
| 8 | Path traversal | `GET /download?file=` | High | `/download?file=../flag_traversal.txt` returns `FLAG{p4th_travers4l_f0und_ya}` |
| 9 | OS command injection | `POST /admin/tools/ping` (requires #7) | Critical | `host=127.0.0.1; whoami` returns `root` |
| 10 | Predictable password reset token | `GET/POST /forgot`, `/reset` | High | Token = base64(md5(username + "salt123")); reusable/derivable, no expiry |
| 11 | Sensitive data exposure — unauthenticated backup endpoint | `GET /backup` | Critical | Returns all usernames + password hashes with no auth |
| 12 | Insecure cookie flags (no HttpOnly) | all cookies | Medium | Enables #4/#5 to actually exfiltrate `document.cookie` |

## Intended attack chain (bonus / "full compromise" exercise)

1. Find reflected or stored **XSS** (#4/#5) — note cookies aren't HttpOnly (#12), so injected JS *can* read `document.cookie`.
2. Alternatively, skip straight to **broken access control** (#7) — no need to actually steal a real admin session; the `role` cookie is trusted as-is, so any authenticated user can just set `role=admin` themselves. (Worth discussing with students: this is even worse than a session-theft-only vuln.)
3. With `role=admin`, reach the **admin ping tool** and get **command injection** (#9) → effectively RCE as the app's OS user.
4. Alternatively/additionally: unauthenticated **`/backup`** (#11) leaks all password hashes → crack weak MD5 hashes (#2) offline → **credential stuffing** back into `/login`.
5. **Path traversal** (#8) is a self-contained flag, independent of the chain — good for partial credit / early win.

## Grading rubric suggestion

- Each of the 12 findings: correctly identified + reproduced = X points
- Report quality (clear repro steps, evidence, correct severity, remediation): X points
- Bonus: demonstrating the full chain (XSS/cookie-forge → admin → RCE): bonus points
- Bonus: cracking the MD5 hashes from `/backup` to recover a plaintext password

## Remediation notes (for report answer-checking)

1. Use parameterized queries / an ORM — never string-format user input into SQL.
2. Use a strong salted hash (bcrypt/argon2/scrypt), never unsalted MD5.
3. Add rate limiting and account lockout/backoff on `/login`.
4/5. Escape all user-controlled output by default (remove `|safe`); use a templating engine's auto-escaping and a Content-Security-Policy.
6. Enforce object-level authorization: check `session['user_id'] == requested_id` (or a proper role check) before returning private data.
7. Never trust a client-editable cookie for authorization; re-verify role server-side from the session/DB on every privileged request.
8. Sanitize/allow-list filenames; resolve the final path and confirm it's still inside the intended directory before opening.
9. Never pass user input to a shell; use subprocess with argument lists (no `shell=True`), or better, avoid shelling out entirely and validate host input against a strict format (e.g. IP regex).
10. Generate reset tokens with a cryptographically random value, store server-side with expiry, and invalidate after use.
11. Remove debug/backup endpoints before any deployment; require strong auth for any endpoint returning user data.
12. Set `HttpOnly`, `Secure`, and `SameSite` on session cookies.

## Reset the environment between student groups

```bash
docker compose down
docker compose up --build
```

This regenerates `vulnbank.db` from scratch (seed users + a fresh path-traversal
flag file), so every group starts from the same clean state.
