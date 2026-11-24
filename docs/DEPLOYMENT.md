# Deployment

## 1. Environments

| Environment | Purpose | Notes |
|---|---|---|
| Local | Development | Local or Atlas free cluster (Atlas needed for transactions, or run a local replica set) |
| Staging | Pre-release testing | Separate Atlas cluster, separate secrets, test email inbox |
| Production | Live | Backups on, alerts on, restricted DB user |

## 2. Recommended Hosting

| Part | Option | Why |
|---|---|---|
| Client | Vercel | Fast CDN, preview deploys per PR |
| Server | Render Web Service (or AWS ECS via Docker) | Supports long-lived WebSocket connections |
| Database | MongoDB Atlas | Managed replica set, backups, transactions |
| Redis (optional) | Upstash or Render Redis | Rate limits, Socket.io adapter, queues |
| Email | Resend, Brevo or SES | Transactional email |
| Errors | Sentry | Client and server |

Serverless functions are not a good fit for the Socket.io server. Use a persistent Node process.

## 3. Production Environment Variables

**Server**

```
NODE_ENV=production
PORT=5000
CLIENT_URL=https://skillswap.example.com
API_ORIGIN=https://api.skillswap.example.com
MONGODB_URI=...
JWT_ACCESS_SECRET=...        # 32+ random bytes
JWT_REFRESH_SECRET=...       # different from access
ACCESS_TOKEN_TTL=15m
REFRESH_TOKEN_TTL=7d
SIGNUP_BONUS_MINUTES=180
DAILY_EARN_CAP_MINUTES=480
SMTP_HOST=...
SMTP_USER=...
SMTP_PASS=...
REDIS_URL=...
SENTRY_DSN=...
COOKIE_DOMAIN=.skillswap.example.com
```

**Client**

```
VITE_API_URL=https://api.skillswap.example.com/api/v1
VITE_SOCKET_URL=https://api.skillswap.example.com
VITE_SENTRY_DSN=...
```

Generate secrets with:

```bash
node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
```

## 4. Cookies Across Domains

Client and API on different subdomains of the same site (for example `app.` and `api.`) work with `SameSite=Lax` and a shared `COOKIE_DOMAIN`. If they are on completely different domains (for example `*.vercel.app` and `*.onrender.com`), cookies become third-party and need `SameSite=None; Secure`, which some browsers restrict. Best practice: use a custom domain with subdomains for both.

## 5. Server Checklist

```js
app.set('trust proxy', 1);

app.get('/health', (_req, res) => res.json({ status: 'ok' }));
app.get('/ready', async (_req, res) => {
  const ok = mongoose.connection.readyState === 1;
  res.status(ok ? 200 : 503).json({ db: ok });
});

process.on('SIGTERM', async () => {
  server.close(async () => {
    await mongoose.connection.close();
    process.exit(0);
  });
});
```

## 6. Docker (optional)

```dockerfile
# server/Dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY src ./src
ENV NODE_ENV=production
EXPOSE 5000
USER node
CMD ["node", "src/server.js"]
```

## 7. CI/CD (GitHub Actions)

```yaml
name: ci
on: [push, pull_request]
jobs:
  server:
    runs-on: ubuntu-latest
    defaults: { run: { working-directory: server } }
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm, cache-dependency-path: server/package-lock.json }
      - run: npm ci
      - run: npm run lint
      - run: npm test
      - run: npm audit --omit=dev --audit-level=high
  client:
    runs-on: ubuntu-latest
    defaults: { run: { working-directory: client } }
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm, cache-dependency-path: client/package-lock.json }
      - run: npm ci
      - run: npm run lint
      - run: npm run build
```

Deploy on merge to `main`: Vercel and Render both auto-deploy from the repo. Use preview deployments for pull requests.

## 8. Database Operations

- Enable Atlas continuous backups and test a restore before launch.
- Create a production application user with read/write only on the `skillswap` database, and no delete on `ledgerEntries` if you enforce that with a custom role.
- Allow-list only your server's IPs (or use private networking/peering).
- Create indexes at deploy time (`autoIndex` off in production, run a migration script instead).
- Monitor slow queries in Atlas Performance Advisor.

## 9. Release Checklist

- [ ] Staging smoke test: sign up, request, accept, complete, review, message
- [ ] Ledger reconciliation job passes on staging data
- [ ] Environment variables set and secrets rotated from dev values
- [ ] Custom domain, HTTPS and cookie settings verified
- [ ] Email sending domain verified (SPF, DKIM)
- [ ] Sentry receiving events from both apps
- [ ] Lighthouse check on landing page (3D scene lazy loaded, reduced motion fallback)
- [ ] Rollback plan: previous deploy can be redeployed from the host dashboard

## 10. Suggested Build Order

1. Auth, profiles, skill taxonomy
2. Wallet and ledger (with tests)
3. Session booking and state machine
4. Matching feed
5. Chat and notifications
6. Reviews and reputation
7. Landing page with GSAP and three.js polish
8. Admin, reports, hardening, launch
