# SkillSwap

> Peer-to-peer skill exchange. Teach what you know, earn credits, spend them learning something new. No money involved.

## Abstract

SkillSwap lets users exchange lessons and mentoring hours without paying money. Credits are earned by teaching and spent by learning. One hour taught earns one hour of credit, so a guitar teacher can learn Python, a designer can learn public speaking, and everyone stays on equal footing.

## Core Features

| Feature | Summary |
|---|---|
| Sign up | Email + password auth, email verification, secure session handling |
| Skill profiles | Skills offered and skills wanted, level, bio, availability, timezone |
| Skill matching | Ranked suggestions based on two-way skill overlap, availability and reputation |
| Session booking | Request, accept, schedule, complete flow with reminders |
| Credit wallet | Ledger-based credits with escrow during a booked session |
| Ratings | Two-way reviews after a completed session |
| Messaging | Real-time chat between matched or booked users |

## Tech Stack

| Layer | Choice |
|---|---|
| Database | MongoDB Atlas (replica set, transactions enabled) |
| Backend | Node.js, Express.js, Mongoose, Zod, Socket.io |
| Frontend | React.js (Vite), React Router, TanStack Query, Zustand |
| UI | Tailwind CSS, React Bits components (copy-paste, TS + Tailwind variant) |
| Animation | GSAP (+ ScrollTrigger, `@gsap/react`), three.js (`@react-three/fiber`) |
| Icons | `@phosphor-icons/react` (**lucide-react is not used**) |
| Optional | Redis (rate limits, socket adapter, queues) |

## Documentation Index

| File | Purpose |
|---|---|
| [RULES.md](docs/RULES.md) | Short, enforceable project rules and Definition of Done |
| [ARCHITECTURE.md](docs/ARCHITECTURE.md) | High-level architecture, layers, modules, data flow |
| [SYSTEM_DESIGN.md](docs/SYSTEM_DESIGN.md) | Requirements, matching, booking state machine, credit ledger, scaling |
| [DATABASE.md](docs/DATABASE.md) | Collections, Mongoose schemas, indexes |
| [API.md](docs/API.md) | REST endpoints and socket events |
| [DESIGN_SYSTEM.md](docs/DESIGN_SYSTEM.md) | Premium minimal light theme: colors, type, spacing, components |
| [UI_COMPONENTS.md](docs/UI_COMPONENTS.md) | React Bits component map per screen, icon rules |
| [ANIMATION.md](docs/ANIMATION.md) | GSAP and three.js motion system, performance and accessibility |
| [SECURITY.md](docs/SECURITY.md) | Threat model, auth, abuse prevention, hardening checklist |
| [FOLDER_STRUCTURE.md](docs/FOLDER_STRUCTURE.md) | Client and server project layout |
| [DEPLOYMENT.md](docs/DEPLOYMENT.md) | Environments, env vars, CI, hosting |

## Quick Start

```bash
# server
cd server
cp .env.example .env
npm install
npm run dev

# client
cd client
cp .env.example .env
npm install
npm run dev
```

## Environment Variables

**server/.env**

```
NODE_ENV=development
PORT=5000
CLIENT_URL=http://localhost:5173
MONGODB_URI=mongodb+srv://<user>:<pass>@<cluster>/skillswap
JWT_ACCESS_SECRET=change-me
JWT_REFRESH_SECRET=change-me-too
ACCESS_TOKEN_TTL=15m
REFRESH_TOKEN_TTL=7d
SIGNUP_BONUS_MINUTES=180
SMTP_HOST=
SMTP_USER=
SMTP_PASS=
REDIS_URL=
```

**client/.env**

```
VITE_API_URL=http://localhost:5000/api/v1
VITE_SOCKET_URL=http://localhost:5000
```

## Design Principles

1. Premium and minimal: generous whitespace, one accent color, restrained motion.
2. Light only: warm off-white canvas, white surfaces, ink-colored text.
3. Trust first: credits are a ledger, every movement is traceable.
4. Fast by default: motion and 3D never block content or hurt Lighthouse scores.
