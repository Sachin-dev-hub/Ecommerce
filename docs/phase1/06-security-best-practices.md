# Phase 1 — Security Best Practices

> File: `docs/phase1/06-security-best-practices.md`
> **Status:** proposed — awaiting confirmation before code is generated.

Security is baked in from day one — retrofitting it later is far harder than designing it in now. Each item below is applied during Phase 1 implementation.

## 1. Authentication & Authorization

| Practice | Implementation |
|----------|----------------|
| Hash passwords | `bcryptjs` with cost factor ≥ 10 — never store plain text or reversible hashes |
| Stateless sessions | JWT with expiry (7 days), signed with a strong `ADMIN_JWT_SECRET` |
| Protect admin routes | Express `auth` middleware verifies token → `req.admin`; every product/order write requires it |
| Never leak secrets | API never returns `passwordHash`; error messages stay generic |

## 2. API & Input Security

| Practice | Implementation |
|----------|----------------|
| Input validation | `express-validator` on all bodies/params — malformed input → 400, never reaches DB |
| SQL injection | None possible — Prisma parameterizes all queries |
| Security headers | `helmet` (CSP, X-Frame-Options, HSTS, etc.) |
| CORS | Restricted to the frontend origin (not `*`) |
| Server-side math | Order totals computed on the server; client-sent totals ignored |
| Rate limiting (note) | Added when exposed publicly (Phase 7); simple in dev |

## 3. File Upload Security

| Practice | Implementation |
|----------|----------------|
| Allow-list MIME types | Only `jpeg/png/webp` accepted |
| Size limit | Max 5 MB per image |
| Safe storage | Multer generates unique filenames (timestamp + random) — never user-supplied names |
| No path traversal | Filenames are generated server-side; served only from `/uploads` |
| Future: S3 | Phase 5 moves images to S3 with bucket policies |

## 4. Secrets Management

- `.env` holds `DATABASE_URL` + `ADMIN_JWT_SECRET` → **gitignored**
- `.env.example` committed with placeholder values
- Never commit `.env`, uploaded images, or `node_modules`
- Phase 4 (Kubernetes) replaces env vars with **Secrets**; Phase 5 with AWS **Secrets Manager**/parameter store

## 5. OWASP Top 10 Mapping (quick reference)

| OWASP | Our control |
|-------|-------------|
| A01 Broken Access Control | JWT auth middleware on all admin routes |
| A02 Cryptographic Failures | bcrypt hashing, JWT over HTTPS (Phase 6), no secrets in code |
| A03 Injection | Prisma parameterized queries + input validation |
| A04 Insecure Design | Server-side totals, stock check inside transaction |
| A05 Security Misconfiguration | Helmet, restricted CORS, no debug stack traces |
| A06 Vulnerable Components | Keep dependencies updated; Trivy scanning in Phase 4 |
| A07 Auth Failures | Strong JWT secret, secure login endpoint |
| A08 Integrity | Multer file-type allow-list |
| A09 Logging | Logged requests for admin actions (Phase 7 centralizes) |
| A10 SSRF | No user-supplied URLs fetched by the server |

## 6. Real-World Notes

- **Development ≠ Production.** Dev uses `http://localhost` and open CORS; production gets HTTPS (Phase 6), locked-down CORS, rate limiting, and centralized logging (Phase 7).
- **Principle of least privilege** — the DB user `diya_user` gets only the privileges the app needs.
- **Never roll your own crypto/auth** — bcrypt + JWT from battle-tested libraries.

See also: [04-api-design.md](04-api-design.md) · [05-implementation-steps.md](05-implementation-steps.md).