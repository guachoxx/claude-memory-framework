# Technical Report: auth-refactor

## Overview
Migration of ShopFast authentication from express-session to JWT with refresh token rotation.

## What Was Built

### Phase 1: JWT Infrastructure
- **`apps/api/src/utils/jwt.js`**: Token utilities
  - `signAccessToken(payload)` — RS256, 15min expiry
  - `signRefreshToken()` — crypto.randomBytes(64), returns raw + hash
  - `verifyAccessToken(token)` — returns decoded payload or throws
  - Keys loaded from `JWT_PRIVATE_KEY` / `JWT_PUBLIC_KEY` env vars

- **`refresh_tokens` table**: id (uuid), user_id (FK), token_hash (varchar 128), expires_at (timestamp), revoked_at (nullable timestamp), created_at

### Phase 2: Auth Endpoints
- **`POST /auth/login`**: Validates credentials, creates refresh token in DB, returns `{ accessToken, refreshToken, expiresIn: 900 }`
- **`POST /auth/refresh`**: Validates refresh token hash against DB, checks not revoked/expired, revokes old token, issues new pair
- **`POST /auth/logout`**: Revokes all refresh tokens for `req.user.id` (sets `revoked_at = now()`)

### Technical Decisions
- **RS256 over HS256**: Allows stateless verification without sharing the private key
- **Refresh token rotation**: Each refresh invalidates the previous token — limits damage from token theft
- **Hash before store**: Refresh tokens stored as bcrypt hashes, never plain text
- **Access token in response body**: Frontend stores in memory (not localStorage) to reduce XSS surface

## Phases 3-4
Pending. See PLAN for details.
