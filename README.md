# apogee-sanctum

API tokens for [Apogee](https://github.com/venomous-maker) apps: personal access
tokens limited to chosen abilities, with expiry, `last_used_at` and revocation,
and session sign-in for the app's own single-page front end.

In an apogee-framework app, importing the module is all it takes:
`SanctumPackage` adds the `sanctum` guard (over the app's own users), the
`abilities:` and `ability:` middleware, the `personal_access_tokens` migration
(run it with `migrate`) and `sanctum:prune-expired`.

```typescript
import "std"
import "apogee-sanctum"
use apogee.*

let t = Sanctum.createToken(user, "deploy script", ["servers:read", "deploy"])
print(t.plainText)                                   // "17|9f2c…": shown once, stored only as a hash

router.middleware(["auth:sanctum", "abilities:deploy"]).group(fn(r: Router) { r.post("/deploy", deploy) })
if (Sanctum.tokenCan(req, "servers:read")) { ... }
Sanctum.revoke(t.token.id)
```

- **Tokens** are `id|secret`: the id finds the row by its primary key and the
  secret is compared as a SHA-256 hash in constant time. `["*"]` grants every
  ability. `expiresAt` (or `Sanctum.expirationMinutes`) ends a token;
  `Sanctum.prune()` removes expired ones.
- **Speed:** a token looked up is kept in memory for `cacheSeconds` (5; revoking
  here drops it at once), and `last_used_at` is written at most once a
  `touchEverySeconds` (60) per token, not on every request.
- **Single-page apps:** a request without a token whose `Origin` (or `Referer`)
  is one of `Sanctum.stateful`'s hosts is signed in by the session cookie,
  through `sessionGuard` ("web"). Such a request may do whatever its user may.
- **Middleware:** `abilities:a,b` needs all of them, `ability:a,b` any one.

## Settings

| Setting | Default |
|---|---|
| `sanctum.expiration` | `SANCTUM_EXPIRATION`, minutes a new token lasts (0: until revoked) |
| `sanctum.stateful` | `SANCTUM_STATEFUL_DOMAINS`, else localhost, 127.0.0.1 and `APP_URL`'s host |
| `sanctum.guard` | `web`: the session guard for the front end |
| `sanctum.cache` | `SANCTUM_CACHE`: `""` (memory only), `default` (the app's cache store) or a store's name |
| `sanctum.cache_seconds` | seconds a token stays in a process's memory: 5, or 1 with a shared cache |
| `sanctum.shared_seconds`, `sanctum.miss_seconds` | 60 (a token in the shared cache), 5 (an unknown id remembered) |
| `sanctum.touch_every` | 60: `last_used_at` written at most this often |

## Caching

A lookup checks this process's memory, then the shared cache (when
`sanctum.cache` names one), then the database. Ids that don't exist are
remembered briefly, so a stream of bad tokens costs no queries. Revoking
removes the token from the shared cache at once; other processes stop
accepting it within `cache_seconds` (1 second with a shared cache). A shared
cache that fails is skipped: the database answers. Costs are measured by
`grav bench bench/tokens.grav`: about 1.3 µs from memory and 4.6 µs from an
in-process shared cache (a Redis round trip adds its network time).

## Stores

`PersonalAccessToken` is an Orbit model (`personal_access_tokens`, made by
`CreatePersonalAccessTokensTable`), kept by `OrbitAccessTokenStore` in the app's
database (SQLite, MySQL/MariaDB or PostgreSQL). `MemoryAccessTokenStore` is for
tests. Outside the framework, set `Sanctum.use(store)`, `Sanctum.provider`,
`Sanctum.auth`, and add `new SanctumGuard("sanctum")` yourself.

## Tests

```sh
grav test tests/sanctum.grav
grav test tests/package.grav
grav test tests/shared.grav
SANCTUM_MYSQL_URL=mysql://… SANCTUM_POSTGRES_URL=postgres://… grav test tests/sanctum.grav
```

## License

MIT
