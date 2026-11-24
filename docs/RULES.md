# Project Rules

Short, enforceable rules for every contributor (and every AI coding tool) working on SkillSwap. If a rule here conflicts with a quick hack, the rule wins.

---

## 1. Code Rules (keep it short)

| Rule | Limit |
|---|---|
| Function | Max ~30 lines, does one thing |
| React component | Max ~150 lines, split when bigger |
| File | Max ~300 lines |
| Function params | Max 3, otherwise pass an object |
| Nesting | Max 3 levels, use early returns |
| Duplication | Same logic twice is fine, **three times means extract it** |

Do:
- Use early returns instead of nested `if/else`.
- Name things by what they do (`holdCredits`, not `process`).
- Keep business logic in services and hooks, never inside JSX or controllers.
- Validate once at the edge (Zod), trust typed data inside.
- Delete dead code and unused imports. No commented-out code.
- Comment only the **why**, never the what.

Don't:
- No copy-paste blocks, no 200-line "god" components or services.
- No magic numbers or strings. Put them in `constants.js` or env.
- No `console.log` in committed code. Use the logger.
- No `lucide-react`. Icons come from `@phosphor-icons/react` only.

---

## 2. Workflow (follow in order)

```
1. Pick ONE task        -> small, shippable in under a day
2. Branch               -> feat/<name>, fix/<name>, docs/<name>
3. Design first         -> 3 lines: input, output, edge cases (check SYSTEM_DESIGN.md)
4. Schema + validation  -> Zod schema before the controller
5. Service logic        -> pure, testable, transaction if credits move
6. Route + controller   -> thin: parse, call service, respond
7. Client               -> hook (TanStack Query) -> reusable UI -> page
8. Test                 -> wallet + state machine always; happy path + 1 failure for the rest
9. Self-review          -> run the Definition of Done below
10. PR + merge          -> small PRs, squash merge, conventional commit message
```

Commit format: `feat: add session request`, `fix: stop double hold`, `docs: update rules`, `chore: bump deps`.

### Definition of Done

- [ ] Input validated (Zod) and ownership checked
- [ ] No duplicated logic, reusable pieces extracted
- [ ] Loading, empty and error states handled in UI
- [ ] Works on mobile width and with keyboard only
- [ ] Reduced-motion respected for any animation
- [ ] No secrets, tokens or personal data in logs or client code
- [ ] Lint passes, tests pass, `npm audit` has no new high issues
- [ ] Docs updated if behavior or API changed

---

## 3. Reusable Components and Helpers

Rule: **build once in the shared folder, import everywhere.** Pages only compose.

### Frontend (`client/src/components/ui`, `hooks`, `lib`)

| Reusable | Replaces |
|---|---|
| `Button`, `Input`, `Textarea`, `Select` | Ad-hoc styled elements |
| `Modal`, `Sheet`, `Toast`, `Tabs`, `Tooltip` | One-off overlays |
| `Card`, `Section`, `Container`, `EmptyState`, `Skeleton` | Repeated layout markup |
| `SkillChip`, `CreditBadge`, `Rating`, `Avatar` | Domain UI repeated on many pages |
| `RevealGroup` (GSAP) | Per-page scroll animation code |
| `useApi` / feature hooks (`useSessions`, `useWallet`) | `fetch` calls inside components |
| `useReducedMotion`, `useMediaQuery` | Repeated window checks |
| `formatCredits`, `formatDate` | Inline formatting |

Component rules:
- Props are small and typed. Use `variant` and `size` props instead of new components.
- Styling comes from design tokens only (see DESIGN_SYSTEM.md). No hardcoded hex colors.
- React Bits components are wrapped once in `ui/`, never used raw across many pages.
- One component, one job. Data fetching lives in hooks, not in presentational components.

### Backend (`server/src/utils`, `middleware`)

| Reusable | Purpose |
|---|---|
| `asyncHandler` | No try/catch in every controller |
| `AppError` | One error shape everywhere |
| `validate(schema)` | One Zod middleware for body, query, params |
| `requireAuth`, `requireRole` | Auth in one place |
| `assertOwner(doc, userId)` | One IDOR guard (section 4) |
| `setAuthCookies`, `clearAuthCookies` | One cookie policy |
| `withTransaction(fn)` | Wallet and session changes |
| `makeBloom(items, rate)` | Fast pre-checks (section 8) |

```js
// utils/asyncHandler.js
export const asyncHandler = (fn) => (req, res, next) => Promise.resolve(fn(req, res, next)).catch(next);
```

---

## 4. Security Rules

### 1) Injection and Input

| Check | Rule | How |
|---|---|---|
| **MongoDB injection** | Query values must be plain strings, numbers or ObjectIds | Strict Zod schemas on every body, query and param. Reject keys starting with `$` or containing `.`. Never pass `req.body` or `req.query` straight into a query |
| **XSS block** | Treat all user text as hostile | React escapes by default. No `dangerouslySetInnerHTML`. Strip HTML server-side from bios and messages. Strict CSP via Helmet |
| **CSRF protection** | Cookies are sent automatically, so protect writes | `SameSite=Lax` cookies + require a custom header on POST/PATCH/PUT/DELETE |
| **File upload validation** | Never trust name or mimetype | Size limit, allow-list types, check magic bytes, random filename, store outside the web root or in cloud storage, no SVG |
| **SSRF audit** | Server must not fetch user-supplied URLs | Avoid the feature. If needed, allow-list hosts and block private IPs |

```js
// middleware/validate.js
export const validate = (schema) => (req, _res, next) => {
  const r = schema.strict().safeParse({ body: req.body, query: req.query, params: req.params });
  if (!r.success) return next(new AppError('VALIDATION_ERROR', 400, r.error.issues));
  Object.assign(req, r.data);
  next();
};
```

```js
// middleware/csrf.js  (use with SameSite cookies)
export const csrfGuard = (req, _res, next) => {
  if (['GET', 'HEAD', 'OPTIONS'].includes(req.method)) return next();
  if (req.get('x-requested-with') !== 'SkillSwap') return next(new AppError('CSRF_BLOCKED', 403));
  next();
};
```

```js
// utils/sanitize.js
import sanitizeHtml from 'sanitize-html';
export const clean = (s) => sanitizeHtml(s, { allowedTags: [], allowedAttributes: {} }).trim();
```

```js
// middleware/upload.js  (avatars only)
import multer from 'multer';
import { fileTypeFromBuffer } from 'file-type';
const ALLOWED = new Set(['image/jpeg', 'image/png', 'image/webp']);
export const upload = multer({ storage: multer.memoryStorage(), limits: { fileSize: 2 * 1024 * 1024, files: 1 } });
export const verifyImage = async (req, _res, next) => {
  const type = req.file && (await fileTypeFromBuffer(req.file.buffer));
  if (!type || !ALLOWED.has(type.mime)) return next(new AppError('INVALID_FILE', 400));
  next();
};
```

SSRF rule of thumb (only if a server-side fetch is ever added):

```js
// utils/safeFetch.js
import dns from 'node:dns/promises';
import ipaddr from 'ipaddr.js';
const HOSTS = new Set(['api.trusted-provider.com']);
export async function safeFetch(url) {
  const u = new URL(url);
  if (u.protocol !== 'https:' || !HOSTS.has(u.hostname)) throw new AppError('URL_BLOCKED', 400);
  const { address } = await dns.lookup(u.hostname);
  if (ipaddr.parse(address).range() !== 'unicast') throw new AppError('URL_BLOCKED', 400);
  return fetch(u, { redirect: 'error', signal: AbortSignal.timeout(5000) });
}
```

SSRF audit checklist: search the codebase for `fetch(`, `axios(`, `http.get(` with user-controlled values. v1 should have none.

### 2) Auth and Access

| Check | Rule | How |
|---|---|---|
| **BOLA / IDOR fix** | Every resource read or write checks the owner | `assertOwner` in the service, and scope queries by `userId` |
| **Rate limiting** | Limit by IP and by account | `express-rate-limit` (+ Redis store when scaled). Limits in API.md |
| **Password hashing** | Never store or log plain passwords | `argon2id` (or bcrypt cost 12+), `select: false` on the hash field |
| **MFA** | Required for admins, optional for users | TOTP (`otplib`), recovery codes, re-verify for sensitive actions |
| **Server-side permissions** | Client hides buttons, server enforces | `requireRole` + policy checks in services. Never trust a role or `userId` from the body |
| **RLS enable** | Row-level isolation | See note below |

```js
// utils/assertOwner.js
export const assertOwner = (doc, userId, field = 'userId') => {
  if (!doc) throw new AppError('NOT_FOUND', 404);
  if (String(doc[field]) !== String(userId)) throw new AppError('NOT_FOUND', 404); // 404 avoids leaking existence
  return doc;
};

// usage: sessions only visible to participants
const s = await Session.findOne({ _id: id, $or: [{ teacherId: me }, { learnerId: me }] });
if (!s) throw new AppError('NOT_FOUND', 404);
```

```js
// MFA (TOTP) check
import { authenticator } from 'otplib';
export const verifyTotp = (token, secret) => authenticator.verify({ token, secret });
```

**RLS note:** MongoDB has **no built-in row-level security** (that is a PostgreSQL/Supabase feature). With MongoDB, get the same effect by:
1. Accessing data only through a repository layer that **always adds the owner filter**.
2. Using a least-privilege database user (no delete on `ledgerEntries`).
3. Writing a test per collection that proves user A cannot read or change user B's data.

If any part moves to PostgreSQL or Supabase, enable RLS on every table and write policies before exposing it.

### 3) Secrets and Tokens

| Check | Rule |
|---|---|
| **Strong JWT secrets** | 48+ random bytes, separate access and refresh secrets, rotate on suspicion |
| **API keys server-side** | Third-party keys live only in server env. Never in `VITE_*` variables or client code |
| **No tokens in localStorage** | Use **HttpOnly cookies**. Nothing sensitive in `localStorage` or `sessionStorage` |
| **Default credentials changed** | No default DB, Redis, admin or seed passwords in any environment |

```bash
node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
```

```js
// utils/cookies.js
const base = { httpOnly: true, secure: process.env.NODE_ENV === 'production', sameSite: 'lax', path: '/' };
export const setAuthCookies = (res, access, refresh) => {
  res.cookie('access_token', access, { ...base, maxAge: 15 * 60 * 1000 });
  res.cookie('refresh_token', refresh, { ...base, maxAge: 7 * 24 * 60 * 60 * 1000, path: '/api/v1/auth' });
};
export const clearAuthCookies = (res) => ['access_token', 'refresh_token'].forEach((c) => res.clearCookie(c, base));
```

Rules:
- `.env` is gitignored. Only `.env.example` with placeholders is committed.
- Seed scripts must read the admin password from env and refuse to run with an empty or default value.
- Run a secret scan (gitleaks) in CI. If a secret ever hits git history, rotate it.

### 4) Config and Hygiene

| Check | Rule | How |
|---|---|---|
| **Strict CORS** | One explicit origin, credentials on | `cors({ origin: CLIENT_URL, credentials: true })`. Never `*` with cookies |
| **Webhook signatures** | Verify before processing | HMAC on the **raw body**, timing-safe compare, reject old timestamps |
| **Hide source maps** | Not public in production | `build.sourcemap: false` (or `'hidden'` and upload to Sentry privately) |
| **Clean production logs** | No secrets or PII | Logger redaction, no stack traces to clients |
| **Dependency updates** | Never let them rot | Dependabot or Renovate, `npm audit` in CI |

```js
// middleware/verifyWebhook.js  (needs express.raw() on the webhook route)
import crypto from 'node:crypto';
export const verifyWebhook = (secret) => (req, _res, next) => {
  const sig = Buffer.from(req.get('x-signature') || '', 'hex');
  const mac = crypto.createHmac('sha256', secret).update(req.body).digest();
  if (sig.length !== mac.length || !crypto.timingSafeEqual(sig, mac)) return next(new AppError('BAD_SIGNATURE', 401));
  next();
};
```

```js
// vite.config.js
export default defineConfig({ build: { sourcemap: false } });
```

```js
// config/logger.js
import pino from 'pino';
export const logger = pino({
  level: process.env.NODE_ENV === 'production' ? 'info' : 'debug',
  redact: ['req.headers.authorization', 'req.headers.cookie', '*.password', '*.token', '*.refreshToken'],
});
```

Production error handler returns `{ code, message }` only. Stack traces go to the logger and Sentry, never to the response.

---

## 5. Validation Rules

1. **One schema per endpoint**, in `<module>.schema.js`, written with Zod.
2. Schemas are `strict()`: unknown keys are rejected, not ignored.
3. Reuse shared pieces (`objectId`, `email`, `pagination`, `password`) from `utils/schemas.js`.
4. Validate **at the edge only** (middleware). Services receive clean data.
5. The same schema powers the client form (React Hook Form + Zod resolver) so rules never drift.
6. Server validation is the source of truth. Client validation is only for UX.

```js
// utils/schemas.js
import { z } from 'zod';
export const objectId = z.string().regex(/^[a-f\d]{24}$/i);
export const email = z.string().trim().toLowerCase().email().max(254);
export const pagination = z.object({ page: z.coerce.number().int().min(1).default(1), limit: z.coerce.number().int().min(1).max(50).default(20) });
```

---

## 6. Bloom Filter Rules (fast validation)

A Bloom filter answers "is this value **possibly** in a big set?" in constant time and tiny memory. It can say yes wrongly (false positive) but **never says no wrongly**.

| Use case | If filter says "no" | If filter says "maybe" |
|---|---|---|
| Email or username already taken | Definitely free, skip the DB lookup | Confirm with the database (unique index stays the real guard) |
| Disposable email domains (anti-farming) | Allow | Reject, or flag for review |
| Breached / common passwords | Allow | Ask the user to choose a different password |
| Blocked IPs, banned device ids | Allow | Block or challenge |

Rules:
- The database unique index is always the final authority. The filter only saves lookups.
- Target false-positive rate: **0.1% or lower** (`0.001`).
- Bloom filters cannot delete items. Rebuild on a schedule (for example nightly) or use a counting Bloom filter if removals matter.
- Load blocklists at boot from a file and keep the filter in memory. With multiple instances, build it the same way on each, or use RedisBloom.
- Normalize before adding or checking (lowercase, trim).
- Never expose the check as an oracle. Return the same generic message and rate limit it so it cannot be used to enumerate accounts.

```js
// utils/bloom.js  (npm i bloom-filters, confirm the API in its docs after install)
import { BloomFilter } from 'bloom-filters';

export const makeBloom = (items, errorRate = 0.001) => {
  const f = BloomFilter.create(Math.max(items.length, 1000), errorRate);
  items.forEach((i) => f.add(String(i).toLowerCase().trim()));
  return { has: (v) => f.has(String(v).toLowerCase().trim()) };
};
```

```js
// usage: registration pre-checks
const disposable = makeBloom(disposableDomains);
const weakPasswords = makeBloom(commonPasswords);

if (disposable.has(email.split('@')[1])) throw new AppError('EMAIL_NOT_ALLOWED', 400);
if (weakPasswords.has(password)) throw new AppError('PASSWORD_TOO_COMMON', 400);
```

---

## 7. Pre-Merge Security Checklist

**Injection and input**
- [ ] Every route has a strict Zod schema
- [ ] No raw `req.body` or `req.query` in a Mongo query
- [ ] User text sanitized, no `dangerouslySetInnerHTML`
- [ ] CSRF guard active on all writes
- [ ] Uploads: size, type allow-list, magic-byte check
- [ ] No server-side fetch of user URLs (or safeFetch used)

**Auth and access**
- [ ] `assertOwner` or scoped query on every resource
- [ ] Rate limits on auth, booking and chat
- [ ] argon2id hashing, no hash in responses
- [ ] MFA available, required for admin
- [ ] Permissions enforced on the server
- [ ] Isolation test: user A cannot touch user B

**Secrets and tokens**
- [ ] JWT secrets 48+ bytes and different
- [ ] No keys in client bundle
- [ ] Tokens only in HttpOnly cookies
- [ ] No default credentials anywhere

**Config and hygiene**
- [ ] CORS limited to the real client origin
- [ ] Webhooks verify signatures
- [ ] Source maps not public
- [ ] Logs redacted, no stack traces to users
- [ ] Dependencies updated, `npm audit` clean