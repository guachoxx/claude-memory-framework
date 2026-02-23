# Plan: auth-refactor

## Phase 1: JWT Infrastructure
- Create `apps/api/src/utils/jwt.js` — sign/verify functions, key loading
- Create Prisma migration for `refresh_tokens` table
- Add `jsonwebtoken` dependency
- **Files**: jwt.js (new), schema.prisma (modify), package.json (modify)

## Phase 2: Auth Endpoints
- Rewrite `POST /auth/login` to return JWT + refresh token
- Create `POST /auth/refresh` for token rotation
- Rewrite `POST /auth/logout` to revoke refresh tokens
- **Files**: routes/auth.js (modify), jwt.js (extend)

## Phase 3: Middleware Migration
- Create `jwtMiddleware.js` with JWT validation
- Update all 12 route files to use new middleware
- Support dual auth (session OR JWT) during transition
- **Files**: jwtMiddleware.js (new), authMiddleware.js (modify), routes/*.js (12 files)

## Phase 4: Cleanup
- Remove express-session, connect-pg-simple dependencies
- Drop `sessions` table
- Remove session config file
- Remove dual-auth support from middleware
- **Files**: config/session.js (delete), authMiddleware.js (delete), package.json (modify), schema.prisma (modify)
