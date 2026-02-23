# Specifications: auth-refactor

## Scope
Migrate ShopFast authentication from session-based (express-session) to JWT with refresh tokens.

## Requirements
1. Access tokens: JWT, 15-minute expiry, signed with RS256
2. Refresh tokens: opaque, stored hashed in DB, 30-day expiry, single-use with rotation
3. Login endpoint returns access + refresh tokens
4. Protected routes validate JWT from Authorization header
5. Refresh endpoint issues new token pair and revokes old refresh token
6. Logout revokes all refresh tokens for the user
7. Existing session-based auth removed after migration

## Acceptance Criteria
- [ ] Login returns `{ accessToken, refreshToken }` with correct expiry
- [ ] Protected routes reject expired/invalid tokens with 401
- [ ] Refresh endpoint rotates tokens correctly
- [ ] Logout invalidates all user sessions
- [ ] No session table queries after migration complete
- [ ] All existing tests pass + new auth tests added

## Out of Scope
- OAuth/social login (separate project)
- API rate limiting (already handled by nginx)
