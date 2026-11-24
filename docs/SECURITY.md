# Security

SkillSwap holds personal profiles, private messages and a credit economy. The two biggest risks are **account takeover** and **credit abuse**. This document covers both, plus standard web hardening.

## 1. Assets and Threats

| Asset | Threats |
|---|---|
| Accounts | Credential stuffing, brute force, phishing, session theft |
| Credit wallet | Double-spend, race conditions, fake-account farming, collusion |
| Messages and profiles | Data leaks, IDOR, scraping, harassment |
| Infrastructure | Secret leaks, dependency compromise, DoS |

## 2. Authentication

| Control | Implementation |
|---|---|
| Password hashing | `argon2id` (or `bcrypt` cost 12 minimum). Never log or return hashes (`select: false`) |
| Password policy | Min 10 characters, block common/breached passwords (zxcvbn or HIBP k-anonymity check) |
| Access token | JWT, 15 min, signed with `JWT_ACCESS_SECRET`, in `HttpOnly; Secure; SameSite=Lax` cookie |
| Refresh token | Random 256-bit value, only the **hash** stored in DB, 7 days, rotated on every use |
| Reuse detection | Each token belongs to a `family`. If a revoked token is reused, revoke the whole family |
| Logout | Revoke refresh token, clear cookies |
| Brute force | Per-IP and per-email limits, account lock for 15 min after 5 failed logins, generic error messages |
| Email verification | Required before requesting or accepting sessions. Tokens are random, stored hashed, expire in 24h |
| Password reset | Single-use hashed token, 30 min expiry, always respond 200, revoke all sessions on reset |
| Enumeration | Same response and similar timing for existing and non-existing emails |

JWT claims: `sub`, `role`, `iat`, `exp`. Pin the algorithm (`HS256`) when verifying and reject `none`.

## 3. Authorization

- Deny by default. Every route declares its auth requirement.
- **Ownership checks in the service layer** for every resource (sessions, conversations, wallet, reviews). Never trust IDs from the client alone. This prevents IDOR.
- Roles: `user`, `admin`. Admin routes need the role and are all written to `auditLogs`.
- Only session participants can read a session, message in its conversation, or review it.
- Blocked users cannot message, request sessions or see each other in matches.

## 4. Input Validation and Injection

| Threat | Defense |
|---|---|
| NoSQL injection | Validate every body, query and param with **Zod** using strict schemas (no raw objects reaching Mongoose queries). Reject keys starting with `$` or containing `.` |
| Mass assignment | Whitelist fields via Zod. Never pass `req.body` straight to `create` or `update` |
| XSS | React escapes by default. Never use `dangerouslySetInnerHTML` on user content. Strip HTML from bios and messages server-side. Strict CSP |
| CSRF | `SameSite` cookies plus a custom header requirement (`X-Requested-With`) or double-submit token for state-changing requests |
| ReDoS | Escape user input used in regex search, or use text indexes |
| Open redirect | Allow-list redirect targets |
| SSRF | No server-side fetching of user-supplied URLs in v1. Validate meeting links by pattern only |

## 5. Transport and Headers

```js
app.use(helmet({
  contentSecurityPolicy: {
    useDefaults: true,
    directives: {
      'default-src': ["'self'"],
      'script-src': ["'self'"],
      'style-src': ["'self'", 'https://fonts.googleapis.com', "'unsafe-inline'"],
      'font-src': ["'self'", 'https://fonts.gstatic.com'],
      'img-src': ["'self'", 'data:', 'https:'],
      'connect-src': ["'self'", process.env.CLIENT_URL, process.env.API_ORIGIN],
      'frame-ancestors': ["'none'"],
    },
  },
}));
app.use(cors({ origin: process.env.CLIENT_URL, credentials: true }));
app.disable('x-powered-by');
app.set('trust proxy', 1); // behind Render / proxy
```

- HTTPS everywhere, HSTS enabled.
- CORS: single explicit origin, never `*` with credentials.
- Limit body size: `express.json({ limit: '10kb' })`.
- Tune the CSP to match the final frontend (fonts, three.js, analytics).

## 6. Credit Wallet Integrity

| Risk | Control |
|---|---|
| Double-spend / race | MongoDB transactions, conditional `findOneAndUpdate` with balance check in the same operation |
| Retried requests | `Idempotency-Key` header stored as unique `idempotencyKey` on ledger entries |
| Tampering | Ledger is append-only, no update/delete code paths, production DB role without delete on `ledgerEntries` |
| Drift | Nightly reconciliation job: `wallet.balance` equals sum of ledger, alert on mismatch |
| Client-supplied amounts | Server calculates credit cost from `durationMin`, never trusts a client amount |
| Admin misuse | `ADJUSTMENT` entries require reason, logged in `auditLogs`, alert on large adjustments |

## 7. Abuse and Credit Farming

Because credits have value, expect attempts to mint them with fake accounts that "teach" each other.

| Mitigation | Detail |
|---|---|
| Verified email required | Plus optional phone verification before earning above a threshold |
| Both-party confirmation | Credits release only if both confirm, or after the dispute window with no dispute |
| Daily earn cap | Default 480 credit minutes per user per day |
| New account limits | First 7 days: max 2 sessions as teacher, lower cap |
| Pair limits | Max 3 sessions between the same two users per week for earning purposes |
| Signup bonus control | One per verified email, rate-limit registrations per IP/device, block disposable email domains |
| Anomaly detection | Flag mutual-only trading rings, instant 5-star mutual reviews, sessions shorter than scheduled, same IP/device on both sides |
| Minimum session evidence | Optional meeting-link join tracking or a short post-session checklist |
| Review integrity | Reviews hidden until both submit or 7 days pass, one per person per session |
| Reporting | In-app report on user, message or session, reviewed by admin |

## 8. Messaging and Safety

- Text only in v1. No file uploads, which removes malware vectors.
- Escape and length-limit messages (2000 chars).
- Rate limit sends (30/min).
- Block and report in every thread.
- Optional keyword filter for contact-info sharing and off-platform solicitation before trust is established.
- Socket.io: authenticate on handshake, authorize each `chat:join` against conversation participants, validate every payload with the same Zod schemas, limit event rate per socket.
- Safety copy in onboarding: do not share passwords, do not pay outside the platform.

## 9. Data Protection and Privacy

| Topic | Approach |
|---|---|
| Data minimization | Collect only what the product needs. Public profile shows name, skills, bio, ratings, not email |
| Encryption | TLS in transit, Atlas encryption at rest |
| PII in logs | Never log passwords, tokens, full emails or message bodies. Mask where needed |
| Retention | See DATABASE.md section 6 |
| User rights | Endpoints to export and delete account data. Block deletion while sessions with held credits are open |
| Compliance | Review obligations under India's DPDP Act 2023 and GDPR if serving EU users. Provide privacy policy and consent at signup |
| Minors | Decide and enforce a minimum age in the terms and signup flow |

This is a design guideline, not legal advice. Get a legal review before launch.

## 10. Rate Limiting

```js
import rateLimit from 'express-rate-limit';

export const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  limit: 10,
  standardHeaders: true,
  legacyHeaders: false,
  keyGenerator: (req) => `${req.ip}:${(req.body?.email || '').toLowerCase()}`,
});
```

Use a Redis store once more than one server instance runs. See limits table in API.md.

## 11. Secrets and Configuration

- Secrets only in environment variables or the host's secret manager. `.env` is gitignored, `.env.example` has placeholders.
- Separate secrets per environment. Rotate on suspected exposure.
- Use a MongoDB user with least privilege, IP allow-list Atlas to your host.
- Access and refresh secrets are different and at least 32 random bytes.

## 12. Supply Chain and Dependencies

- Commit lockfiles, run `npm audit` in CI, enable Dependabot or Renovate.
- Prefer well-maintained packages. Review new dependencies.
- Pin GitHub Actions to versions. Use 2FA on GitHub, npm and hosting accounts.
- Because React Bits components are copied into the repo, review the source of what you paste in.

## 13. Logging, Monitoring, Incident Response

- Log auth events (login success/failure, password reset, token reuse), admin actions, credit anomalies, 4xx/5xx spikes.
- Alerts: repeated token-reuse detection, ledger reconciliation failure, spike in registrations from one IP range.
- Incident steps: contain (revoke tokens, suspend accounts), assess, fix, notify affected users when required, write a short post-mortem.

## 14. Pre-Launch Checklist

- [ ] HTTPS and HSTS enabled
- [ ] Helmet, CORS, CSP configured and tested
- [ ] All inputs validated with Zod, no raw `req.body` to Mongoose
- [ ] Ownership checks covered by tests for every resource
- [ ] Rate limits on auth, booking, chat
- [ ] Refresh rotation and reuse detection tested
- [ ] Ledger transactions, idempotency and reconciliation tested
- [ ] Anti-farming caps active
- [ ] No secrets in repo history
- [ ] `npm audit` clean or exceptions documented
- [ ] Error responses do not leak stack traces in production
- [ ] Backups enabled and a restore tested
- [ ] Privacy policy, terms and report/blocking flows live
- [ ] Basic penetration pass against OWASP Top 10
