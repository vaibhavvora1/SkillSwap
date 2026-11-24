# API Reference

Base URL: `/api/v1`. JSON only. Auth uses HttpOnly cookies (`access_token`, `refresh_token`).

## Conventions

**Success**

```json
{ "success": true, "data": { }, "meta": { "page": 1, "limit": 20, "total": 120 } }
```

**Error**

```json
{ "success": false, "error": { "code": "VALIDATION_ERROR", "message": "Invalid input", "details": [{ "path": "email", "message": "Invalid email" }] } }
```

| Status | Meaning |
|---|---|
| 400 | Validation error |
| 401 | Not authenticated |
| 402 | Insufficient credits |
| 403 | Not allowed |
| 404 | Not found |
| 409 | Conflict (state transition, duplicate) |
| 429 | Rate limited |

Pagination: `?page=1&limit=20` (max limit 50). Mutating requests that move credits require the header `Idempotency-Key`.

## Auth

| Method | Path | Description |
|---|---|---|
| POST | `/auth/register` | Create account, wallet and signup bonus |
| POST | `/auth/login` | Sign in, sets cookies |
| POST | `/auth/refresh` | Rotate refresh token |
| POST | `/auth/logout` | Revoke current refresh token |
| POST | `/auth/verify-email` | Verify with token |
| POST | `/auth/forgot-password` | Send reset email (always returns 200) |
| POST | `/auth/reset-password` | Set new password with token |
| GET | `/auth/me` | Current user |

## Profile and Skills

| Method | Path | Description |
|---|---|---|
| GET | `/profiles/:userId` | Public profile |
| PATCH | `/profiles/me` | Update own profile |
| PUT | `/profiles/me/skills` | Replace offered and wanted skills |
| PUT | `/profiles/me/availability` | Replace weekly availability |
| GET | `/skills?q=react` | Search skills |
| POST | `/skills` | Suggest a new skill (pending approval) |

## Matching

| Method | Path | Description |
|---|---|---|
| GET | `/matches` | Ranked recommendations for current user |
| GET | `/matches/search?skill=&level=&day=` | Filtered discovery |

Sample item:

```json
{
  "user": { "id": "...", "name": "Aarav", "avatarUrl": "..." },
  "score": 87,
  "theyTeach": [{ "skill": "Python", "level": "advanced" }],
  "theyWant": [{ "skill": "Guitar" }],
  "mutual": true,
  "ratingAvg": 4.8
}
```

## Sessions

| Method | Path | Description |
|---|---|---|
| POST | `/sessions` | Request a session (holds credits) |
| GET | `/sessions?status=&role=` | My sessions |
| GET | `/sessions/:id` | Session detail |
| POST | `/sessions/:id/accept` | Teacher accepts |
| POST | `/sessions/:id/decline` | Teacher declines |
| POST | `/sessions/:id/cancel` | Cancel with reason |
| POST | `/sessions/:id/confirm` | Confirm it happened |
| POST | `/sessions/:id/dispute` | Open a dispute |

Request body for `POST /sessions`:

```json
{ "teacherId": "...", "skillId": "...", "startsAt": "2026-12-01T10:30:00Z", "durationMin": 60, "message": "Want to learn hooks" }
```

## Wallet

| Method | Path | Description |
|---|---|---|
| GET | `/wallet` | Balance, held, available |
| GET | `/wallet/ledger?page=` | Transaction history |

## Reviews

| Method | Path | Description |
|---|---|---|
| POST | `/sessions/:id/reviews` | Submit review |
| GET | `/users/:id/reviews` | Visible reviews for a user |

## Chat

| Method | Path | Description |
|---|---|---|
| GET | `/conversations` | My conversations |
| GET | `/conversations/:id/messages?before=` | Message history |
| POST | `/conversations/:id/messages` | Send message (REST fallback) |
| POST | `/conversations/:id/read` | Mark read |

## Notifications

| Method | Path | Description |
|---|---|---|
| GET | `/notifications` | List |
| POST | `/notifications/read-all` | Mark all read |

## Safety

| Method | Path | Description |
|---|---|---|
| POST | `/users/:id/block` | Block user |
| DELETE | `/users/:id/block` | Unblock |
| POST | `/reports` | Report user, message or session |

## Admin (role `admin`)

| Method | Path | Description |
|---|---|---|
| GET | `/admin/reports` | Moderation queue |
| POST | `/admin/sessions/:id/resolve` | Resolve dispute |
| POST | `/admin/users/:id/suspend` | Suspend user |
| POST | `/admin/wallet/:userId/adjust` | Ledger adjustment with reason |
| PATCH | `/admin/skills/:id` | Approve or merge skills |

## Socket.io Events

Connection: `io(SOCKET_URL, { withCredentials: true })`. Auth is verified on handshake.

| Direction | Event | Payload |
|---|---|---|
| client to server | `chat:join` | `{ conversationId }` |
| client to server | `chat:send` | `{ conversationId, text, clientId }` |
| client to server | `chat:typing` | `{ conversationId, isTyping }` |
| client to server | `chat:read` | `{ conversationId }` |
| server to client | `chat:message` | `{ message }` |
| server to client | `chat:typing` | `{ conversationId, userId, isTyping }` |
| server to client | `session:updated` | `{ sessionId, status }` |
| server to client | `wallet:updated` | `{ balance, held }` |
| server to client | `notification:new` | `{ notification }` |

## Rate Limits

| Scope | Limit |
|---|---|
| Global per IP | 300 requests / 15 min |
| `/auth/login`, `/auth/register` | 10 / 15 min per IP + per email |
| `/auth/forgot-password` | 5 / hour per IP |
| `POST /sessions` | 20 / hour per user |
| Chat messages | 30 / minute per user |
