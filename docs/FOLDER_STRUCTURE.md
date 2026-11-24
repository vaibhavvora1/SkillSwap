# Folder Structure

Monorepo with two apps. Keep them independently installable and deployable.

```
skillswap/
├── README.md
├── docs/                      # all .md documentation
├── client/
└── server/
```

## Client (React + Vite)

```
client/
├── index.html
├── package.json
├── vite.config.js
├── tailwind.config.js
├── .env.example
├── public/
│   └── favicon.svg
└── src/
    ├── main.jsx
    ├── App.jsx
    ├── routes/
    │   ├── index.jsx                 # route table, lazy loading
    │   └── ProtectedRoute.jsx
    ├── styles/
    │   ├── index.css                 # Tailwind layers
    │   └── tokens.css                # design tokens (DESIGN_SYSTEM.md)
    ├── lib/
    │   ├── api.js                    # fetch/axios client with refresh handling
    │   ├── socket.js                 # Socket.io client
    │   ├── gsap.js                   # plugin registration, reduced-motion helper
    │   ├── motion.js                 # duration, ease, stagger tokens
    │   ├── queryClient.js
    │   └── format.js                 # formatCredits, dates, timezones
    ├── store/
    │   ├── authStore.js              # Zustand
    │   └── uiStore.js
    ├── components/
    │   ├── reactbits/                # copied React Bits components (kept close to upstream)
    │   ├── ui/                       # Button, Input, Modal, SkillChip, CreditBadge, ...
    │   ├── layout/                   # Navbar, Footer, AppShell, Container
    │   ├── three/
    │   │   └── ExchangeScene.jsx     # lazy-loaded hero scene
    │   └── motion/
    │       └── RevealGroup.jsx
    ├── features/
    │   ├── auth/
    │   │   ├── api.js
    │   │   ├── hooks.js
    │   │   ├── LoginForm.jsx
    │   │   └── RegisterForm.jsx
    │   ├── profile/
    │   ├── onboarding/
    │   ├── matching/
    │   ├── sessions/
    │   ├── wallet/
    │   ├── reviews/
    │   ├── chat/
    │   └── notifications/
    ├── pages/
    │   ├── Landing.jsx
    │   ├── Login.jsx
    │   ├── Register.jsx
    │   ├── Onboarding.jsx
    │   ├── Dashboard.jsx
    │   ├── Discover.jsx
    │   ├── Profile.jsx
    │   ├── Sessions.jsx
    │   ├── Wallet.jsx
    │   ├── Messages.jsx
    │   ├── Settings.jsx
    │   └── NotFound.jsx
    └── hooks/
        ├── useReducedMotion.js
        ├── useMediaQuery.js
        └── useSocketEvent.js
```

## Server (Node + Express)

```
server/
├── package.json
├── .env.example
└── src/
    ├── server.js                     # HTTP + Socket.io bootstrap, graceful shutdown
    ├── app.js                        # Express app, middleware, routes
    ├── config/
    │   ├── env.js                    # validated env (Zod)
    │   ├── db.js
    │   └── logger.js                 # pino
    ├── middleware/
    │   ├── auth.js                   # verify access token, attach req.user
    │   ├── requireRole.js
    │   ├── validate.js               # Zod wrapper for body/query/params
    │   ├── rateLimit.js
    │   ├── idempotency.js
    │   ├── error.js                  # central error handler
    │   └── requestId.js
    ├── modules/
    │   ├── auth/
    │   │   ├── auth.routes.js
    │   │   ├── auth.controller.js
    │   │   ├── auth.service.js
    │   │   ├── auth.schema.js        # Zod
    │   │   └── refreshToken.model.js
    │   ├── users/
    │   ├── skills/
    │   ├── matching/
    │   │   ├── matching.service.js   # scoring logic
    │   │   └── ...
    │   ├── sessions/
    │   │   ├── session.stateMachine.js
    │   │   └── ...
    │   ├── wallet/
    │   │   ├── wallet.service.js     # holds, releases, refunds in transactions
    │   │   ├── ledger.model.js
    │   │   └── ...
    │   ├── reviews/
    │   ├── chat/
    │   │   ├── chat.socket.js
    │   │   └── ...
    │   ├── notifications/
    │   ├── reports/
    │   └── admin/
    ├── jobs/
    │   ├── index.js
    │   ├── sessionReminders.job.js
    │   ├── expireRequests.job.js
    │   ├── autoComplete.job.js
    │   └── ledgerReconcile.job.js
    ├── utils/
    │   ├── AppError.js
    │   ├── asyncHandler.js
    │   ├── tokens.js
    │   └── time.js
    └── tests/
        ├── wallet.test.js
        ├── sessions.test.js
        └── auth.test.js
```

## Conventions

| Topic | Rule |
|---|---|
| Naming | `camelCase` for variables, `PascalCase` for components and models, `kebab-case` for route paths |
| Imports | Features import from `components`, `lib`, `store`, never from other features |
| Server layering | routes → controllers → services → models. Services own transactions |
| Env | Validate with Zod at boot, fail fast if missing |
| Tests | Wallet and state machine have the highest coverage priority |
| Commits | Conventional commits (`feat:`, `fix:`, `docs:`) |
| Icons | Only `@phosphor-icons/react`, lint blocks `lucide-react` |
