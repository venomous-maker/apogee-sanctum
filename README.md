# apogee-sanctum

API tokens for [Apogee](https://github.com/venomous-maker) apps: personal access
tokens limited to chosen abilities, with expiry, `last_used_at` and revocation,
and session sign-in for the app's own single-page front end.

In an apogee-framework app, importing the module is all it takes:
`SanctumPackage` adds the `sanctum` guard (over the app's own users), the
`abilities:` and `ability:` middleware, the `personal_access_tokens` migration
(run it with `migrate`) and `sanctum:prune-expired`.

```grav
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
| `sanctum.touch_every`, `sanctum.cache_seconds` | 60, 5 |

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
SANCTUM_MYSQL_URL=mysql://… SANCTUM_POSTGRES_URL=postgres://… grav test tests/sanctum.grav
```

## License

MIT
