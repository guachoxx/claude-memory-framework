# Technical Analysis: auth-refactor

## Current Auth Flow
1. `POST /auth/login` → validates credentials → creates session via `req.session.userId = user.id`
2. `authMiddleware.js` → checks `req.session.userId` → attaches `req.user`
3. `requireRole(role)` → checks `req.user.role`
4. `POST /auth/logout` → `req.session.destroy()`

## Affected Files
- `apps/api/src/middleware/authMiddleware.js` — session check → JWT validation
- `apps/api/src/routes/auth.js` — login/logout/register endpoints
- `apps/api/src/routes/*.js` — 12 route files use `authMiddleware`
- `apps/api/src/config/session.js` — session config (will be removed)
- `apps/api/package.json` — add jsonwebtoken, remove express-session

## Database Changes
- New table: `refresh_tokens` (id, user_id, token_hash, expires_at, revoked_at, created_at)
- `sessions` table: keep during transition, drop in Phase 4

## Constraints
- Must support both session and JWT during transition (Phase 3)
- RS256 requires key pair — store in environment variables
- Refresh token rotation: each use invalidates the old token and issues a new one

## Design Decisions
- RS256 over HS256: allows frontend to verify tokens without sharing the secret
- Refresh tokens stored hashed (bcrypt) rather than plain text
- Access token in memory (not localStorage) to reduce XSS risk
