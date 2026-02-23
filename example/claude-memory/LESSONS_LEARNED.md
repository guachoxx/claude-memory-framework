# Lessons Learned

## Auth / Security
- JWT secret rotation requires invalidating all existing tokens — plan a grace period or use short-lived access tokens + refresh tokens
- Always hash refresh tokens before storing in DB (same as passwords)

## Database
- Prisma migrations must be reviewed before applying — auto-generated migrations can drop columns unexpectedly
