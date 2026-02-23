# Changelog: auth-refactor

## 2026-02-20 — Phase 2: Auth Endpoints
- Rewrote `POST /auth/login` to return `{ accessToken, refreshToken }` (routes/auth.js)
- Created `POST /auth/refresh` with token rotation and old token revocation (routes/auth.js)
- Rewrote `POST /auth/logout` to revoke all refresh tokens for the user (routes/auth.js)
- Added refresh token hashing before DB storage (utils/jwt.js)

## 2026-02-18 — Phase 1: JWT Infrastructure
- Created `apps/api/src/utils/jwt.js` with `signAccessToken()`, `signRefreshToken()`, `verifyAccessToken()`
- Created Prisma migration `20260218_add_refresh_tokens` — new `refresh_tokens` table
- Added `jsonwebtoken@9.0.2` dependency
- Generated RS256 key pair, documented in CREDENTIALS
