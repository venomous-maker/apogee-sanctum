# Changelog

Every release of apogee-sanctum, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

## 0.1.0 (2026-11-25)

- Personal access tokens (hashed, with abilities, expiry, last_used_at and
  revocation), `SanctumGuard` with the session fallback for the front end, and
  the abilities: and ability: middleware.
- `PersonalAccessToken` as an Orbit model with its migration; an Orbit store
  (SQLite, MySQL, PostgreSQL) and a memory one.
- A shared cache (`sanctum.cache`: the app's cache store or a named one)
  behind the in-memory one: tokens are found there by every process, revoking
  removes them from it, unknown ids are remembered briefly, and a failing cache
  is skipped. Cached as tab-separated text (a tenth of JSON's cost).
- Tokens in memory are bounded (`sanctum.memory_tokens`, std's `LruCache`);
  needs grav 1.13.0 or newer.
- `SanctumPackage`: connects itself to an apogee-framework app (settings, the
  guard, the middleware, the migration, `sanctum:prune-expired`).

### Dependencies

- Uses apogee-http 0.3.0, apogee-routing 0.3.0, apogee-auth 0.2.2, apogee-console 0.2.2, apogee-package 0.1.0, grav-orbit 0.3.0, grav-cache 0.2.2, apogee-session 0.2.2, apogee-testing 0.1.1 (from their GitHub tags).
