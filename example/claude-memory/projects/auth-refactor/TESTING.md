# Testing: auth-refactor

## Test Scenarios (from SPECIFICATIONS acceptance criteria)

### AC-1: Login returns tokens
- **Scenario**: POST /auth/login with valid credentials
- **Expected**: 200 with `{ accessToken, refreshToken, expiresIn: 900 }`
- **Result**: PASS — tokens returned, accessToken is valid JWT with RS256

### AC-2: Protected routes reject invalid tokens
- **Scenario**: GET /api/products with expired JWT
- **Expected**: 401 Unauthorized
- **Result**: PENDING — middleware not yet implemented (Phase 3)

### AC-3: Refresh rotates tokens
- **Scenario**: POST /auth/refresh with valid refresh token
- **Expected**: New token pair returned, old refresh token revoked in DB
- **Result**: PASS — verified old token hash marked with `revoked_at`

### AC-4: Logout invalidates all sessions
- **Scenario**: POST /auth/logout, then attempt refresh with any of the user's tokens
- **Expected**: All refresh tokens for user revoked, subsequent refresh returns 401
- **Result**: PASS — confirmed all rows for user_id have `revoked_at` set

## Edge Cases

- Refresh with already-revoked token → 401 (PASS)
- Refresh with expired token → 401 (PASS)
- Login with wrong password → 401 with generic "Invalid credentials" message (PASS)
- Concurrent refresh requests with same token → only first succeeds, second gets 401 (PASS)

## Pending (Phase 3-4)

- [ ] AC-2: Middleware JWT validation (blocked on Phase 3)
- [ ] AC-5: No session table queries after cleanup (Phase 4)
- [ ] Load test: 100 concurrent login requests
- [ ] Verify access token cannot be used after 15-minute expiry
