# Changelog

Every release of apogee-sanctum, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

- Personal access tokens (hashed, with abilities, expiry, last_used_at and
  revocation), `SanctumGuard` with the session fallback for the front end, and
  the abilities: and ability: middleware.
- `PersonalAccessToken` as an Orbit model with its migration; an Orbit store
  (SQLite, MySQL, PostgreSQL) and a memory one.
- `SanctumPackage`: connects itself to an apogee-framework app (settings, the
  guard, the middleware, the migration, `sanctum:prune-expired`).
