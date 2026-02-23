# Architecture

## Stack
- **Runtime**: Node.js 20 LTS
- **API**: Express 4.18, REST, JSON
- **Database**: PostgreSQL 16 (Prisma ORM)
- **Frontend**: React 18, Vite, TanStack Query
- **Auth**: express-session + connect-pg-simple (migrating to JWT)

## Data Model (auth-related)
- `users` — id, email, password_hash, role, created_at
- `sessions` — sid, sess (JSON), expire (managed by connect-pg-simple)
- `refresh_tokens` — id, user_id, token_hash, expires_at, revoked_at (new, Phase 2)

## Key Patterns
- All routes go through `authMiddleware.js` which checks `req.session.userId`
- Role-based access via `requireRole('admin')` middleware
- Password hashing: bcrypt with 12 rounds
