# Current Status: auth-refactor

**Last updated**: 2026-02-20
**Last session summary**: Completed Phase 2 — auth endpoints rewritten to use JWT. Login, refresh, and logout all working with token rotation.
**Next step**: Create `jwtMiddleware.js` with Bearer token extraction and RS256 verification. Wire it into `authMiddleware.js` as dual-auth (accept session OR JWT).
**Done when**: `GET /api/products` returns 200 with a valid JWT and 401 with an expired/invalid token.

## Done
- Phase 1: JWT infrastructure (jwt.js, refresh_tokens migration, dependencies)
- Phase 2: Auth endpoints (login returns tokens, refresh rotates, logout revokes)

## In Progress
- Phase 3: Middleware migration (not started yet)

## Next Steps
- [ ] Create jwtMiddleware.js
- [ ] Add dual-auth support to authMiddleware.js
- [ ] Update 12 route files to pass through new middleware
- [ ] Phase 4: Cleanup (remove session deps, drop table)

## Blockers
- None
