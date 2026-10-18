# 01 — Getting started

This guide gets a licensed Flink Manager instance up and running in a few minutes using the
prebuilt Docker image. For other install paths (release bundle, systemd), see
[02 — Installation](02-installation.md).

> ⚠️ **Licence reminder:** Flink Manager is commercial software. Ensure you have a valid licence
> from [vinteros-labs.com](https://vinteros-labs.com/) before installing. See
> [LICENSE-INFO.md](../LICENSE-INFO.md).

## 1. Requirements

| Requirement | Minimum | Notes |
|---|---|---|
| Docker | any recent version | Recommended way to run Flink Manager |
| Java | **22+** (JDK) | Only needed when running the release bundle directly |
| PostgreSQL | 14+ | For persistent deployments. **Not** needed for a quick test (in-memory profile) |
| Apache Flink cluster | 2.x | The cluster you want to manage |
| Flink SQL Gateway | 2.x (optional) | Needed only for the SQL workbench features |

## 2. Quick test without a database

For a first look, run Flink Manager with the **in-memory profile**: no external database is
needed, and data is kept in memory (lost on restart). This is for quick tests only — for
production, use the default profile with PostgreSQL (see
[02 — Installation](02-installation.md#a-docker-compose)).

Create a `.env` file next to your `docker-compose.yml`:

```dotenv
# .env — the compose file refuses to start without these
DB_PASSWORD=change-me
JWT_SECRET=<random-base64-secret>
AUTH_SECRET_KEY=<random-32-bytes-base64>
```

```yaml
# docker-compose.yml
services:
  flink-manager:
    image: vinteroslabs/flink-manager:latest
    pull_policy: always
    ports:
      - "${SERVER_PORT:-8080}:${SERVER_PORT:-8080}"
    environment:
      FLINK_SERVER_URL: "${FLINK_SERVER_URL:-http://jobmanager:8081}"
      FLINK_SQL_GATEWAY_URL: "${FLINK_SQL_GATEWAY_URL:-http://jobmanager:8083}"
      FLINK_SAVEPOINT_DIR: "${FLINK_SAVEPOINT_DIR:-/tmp/flink-savepoints}"
      DB_URL: "${DB_URL:-jdbc:postgresql://db:5432/flink-manager}"
      DB_USERNAME: "${DB_USERNAME:-flinkmgr}"
      DB_PASSWORD: "${DB_PASSWORD:?Set DB_PASSWORD in .env}"
      JWT_SECRET: "${JWT_SECRET:?Set JWT_SECRET in .env}"
      JWT_EXPIRATION_MS: "${JWT_EXPIRATION_MS:-43200000}"
      AUTH_SECRET_KEY: "${AUTH_SECRET_KEY:?Set AUTH_SECRET_KEY in .env}"
      AUTH_UI_BASE_URL: "${AUTH_UI_BASE_URL:-http://localhost:8080/flink-manager}"
      CORS_ALLOWED_ORIGINS: "${CORS_ALLOWED_ORIGINS:-}"
      SERVER_PORT: "${SERVER_PORT:-8080}"
      SPRING_PROFILES_ACTIVE: "${SPRING_PROFILES_ACTIVE:-in-memory-db}"
```

```bash
docker compose up -d
```

Notes:

- `SPRING_PROFILES_ACTIVE` defaults to `in-memory-db` in this template — no PostgreSQL needed.
  Data is **not persisted**: everything is lost when the container restarts. This profile is
  for quick tests only, not recommended beyond a first look.
- The database variables (`DB_URL`, `DB_USERNAME`, `DB_PASSWORD`) are unused by the in-memory
  profile but are kept in the template so switching to a persistent deployment later is just a
  profile change.
- `CORS_ALLOWED_ORIGINS` can stay empty: the console is served by the application itself, so
  the browser talks to the API on the same origin and no cross-origin permission is needed.
  Set it only if you serve the console from a different origin (e.g. behind a reverse proxy on
  another domain) — see [03 — Configuration](03-configuration.md).
- To make this same compose file production-ready, set `SPRING_PROFILES_ACTIVE=default` in
  `.env` and point `DB_URL` at your PostgreSQL — see
  [02 — Installation](02-installation.md#a-docker-compose).

## 3. Log in

Open **`http://localhost:8080/flink-manager/`** and sign in with the seeded administrator
account:

| Username | Password | Role |
|---|---|---|
| `admin` | `admin` | Full administrative access |

> 🔒 **Change the default password immediately** after first login
> (*Configuration → Users & Roles*). The `operator`, `developer` and `viewer` demo accounts are
> also seeded and should be removed or repurposed before going to production.

## 4. Point it at your Flink cluster

The `FLINK_SERVER_URL` passed at startup is only a bootstrap default. You can review or change
the cluster connection at any time in the UI under *Configuration → Flink Cluster* — values
saved there take effect at runtime, without a restart.

If you plan to use the SQL workbench, also set the SQL Gateway endpoint
(`FLINK_SQL_GATEWAY_URL` or the same page).

## 5. Activate your licence

Go to *Configuration → License* and paste the licence token you received from Vinteros Labs.
Until a valid licence is installed, protected operations are locked. See
[04 — Licensing](04-licensing.md).

## 6. Verify

| Check | URL |
|---|---|
| Console | `http://localhost:8080/flink-manager/` |
| Health endpoint | `http://localhost:8080/actuator/health` |
| REST API base | `http://localhost:8080/api/flink` |
| API docs (Swagger) | `http://localhost:8080/swagger-ui.html` |

## Next steps

- Production install (Docker, bundle + systemd) → [Installation](02-installation.md)
- All environment variables → [Configuration](03-configuration.md)
- Licence activation details → [Licensing](04-licensing.md)

## Help & resources

- Online documentation: [vinteros-labs.com/resources/docs/](https://vinteros-labs.com/resources/docs/)
- Changelog: [vinteros-labs.com/resources/changelog/](https://vinteros-labs.com/resources/changelog/)
- Blog: [vinteros-labs.com/resources/blog/](https://vinteros-labs.com/resources/blog/)
- Something not working? [Open a GitHub issue](https://github.com/vinteroslabs/flink-manager-doc/issues)
  or create a ticket on [vinteros-labs.com](https://vinteros-labs.com/).
