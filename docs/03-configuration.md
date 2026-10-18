# 03 — Configuration

Flink Manager is configured with **environment variables**. The same variables work in the
Docker image, the release bundle scripts (`flink-manager.sh`, `flink-manager.bat`) and the
systemd unit. Defaults shown match the shipped release.

> Values saved through the console (*Configuration → Flink Cluster*) override the bootstrap
> `FLINK_*` variables at runtime, without a restart.

## Server

| Variable | Default | Description |
|---|---|---|
| `SERVER_PORT` | `8080` | HTTP port of the application (console + REST API) |

## Database (PostgreSQL)

| Variable | Default | Description |
|---|---|---|
| `DB_URL` | `jdbc:postgresql://localhost:5432/flink-manager` | JDBC URL. The database must exist; the schema is created/migrated automatically on startup. Unused by the `in-memory-db` profile |
| `DB_USERNAME` | `postgres` | Database user |
| `DB_PASSWORD` | *(empty)* | Database password — **always set in production** |

## Flink cluster

| Variable | Default | Description |
|---|---|---|
| `FLINK_SERVER_URL` | `http://localhost:8081` | Flink JobManager REST endpoint (bootstrap default) |
| `FLINK_SQL_GATEWAY_URL` | `http://localhost:8083` | Flink SQL Gateway REST endpoint (bootstrap default). Leave unset to disable the SQL features |
| `FLINK_SAVEPOINT_DIR` | `/tmp/flink-savepoints` | Savepoint directory used only when the cluster defines none. Must be writable by the JobManager **and** TaskManagers |

## Security

| Variable | Default | Description |
|---|---|---|
| `JWT_SECRET` | dev-only value | Base64 HMAC secret for session tokens — **override in production** |
| `JWT_EXPIRATION_MS` | `43200000` (12 h) | Session token validity |
| `AUTH_SECRET_KEY` | dev-only value | Base64 AES-256-GCM key used to encrypt authentication-provider secrets at rest — **override in production** |
| `AUTH_UI_BASE_URL` | `http://localhost:8080/flink-manager` | Base URL of the console, used for post-login redirects (relevant behind a reverse proxy) |
| `CORS_ALLOWED_ORIGINS` | `http://localhost:3000` | Comma-separated list of allowed origins. **Leave empty** when the console is served by the application itself (the browser then makes same-origin API calls, which need no CORS approval). Set it only when the console is reachable from a different origin than the API (e.g. behind a reverse proxy on another domain). Do not set it to an empty *string* in a way that yields a blank entry — unset the variable instead |

## TLS (inbound HTTPS)

Flink Manager can serve HTTPS itself. Plain HTTPS needs only a keystore; the truststore is
required only for mutual TLS (client certificates).

| Variable | Default | Description |
|---|---|---|
| `SSL_ENABLED` | `false` | Enable inbound HTTPS |
| `SSL_KEYSTORE_PATH` | — | Keystore with server certificate + private key (PKCS12, JKS or JCEKS; type auto-detected when `SSL_KEYSTORE_TYPE` is blank) |
| `SSL_KEYSTORE_PASSWORD` | — | Keystore password |
| `SSL_KEYSTORE_TYPE` | *(auto)* | Optional explicit type: `PKCS12`, `JKS`, `JCEKS` |
| `SSL_KEY_PASSWORD` | *(= keystore password)* | Password of the private key inside the keystore |
| `SSL_TRUSTSTORE_PATH` | — | Truststore, only for mutual TLS |
| `SSL_TRUSTSTORE_PASSWORD` | — | Truststore password |
| `SSL_TRUSTSTORE_TYPE` | *(auto)* | Optional explicit truststore type |
| `SSL_CLIENT_AUTH` | `NONE` | `NONE` = plain HTTPS · `WANT` = request client cert, tolerate absence · `NEED` = require client cert |
| `SSL_ENABLED_PROTOCOLS` | *(container defaults)* | Optional comma-separated list, e.g. `TLSv1.3,TLSv1.2` |

> Trusting the certificates of the Flink REST API / SQL Gateway is a separate, outbound
> concern — configure it in the console on *Configuration → Flink Cluster*.

## JVM & logging

| Variable | Default | Description |
|---|---|---|
| `JVM_OPTS` | `-Xms512m -Xmx1g -XX:+UseG1GC …` | JVM flags. Defaults include G1 GC, a metaspace cap, and heap dump + fast exit on OOM |
| `LOG_DIR` | `./logs` (bundle) / `/opt/flink-manager/logs` (image) | Where log files are written |
| `LOG_FILE_NAME` | `flink-manager.log` | Rotated log file name |
| `LOG_LEVEL` | `INFO` | Root log level |
| `LOG_MAX_FILE_SIZE` | `10MB` | Size per rotated file |
| `LOG_MAX_HISTORY` | `30` | Rotated files retained |
| `LOG_TOTAL_SIZE_CAP` | `500MB` | Total size cap of rotated logs |

## Spring profiles

The application honours `SPRING_PROFILES_ACTIVE`. **You normally don't need to set it** — the
default profile (PostgreSQL) is what production uses. The only reason to set it is a quick test
without a database:

| Profile | Storage | Use |
|---|---|---|
| `default` | PostgreSQL (`DB_URL` / `DB_USERNAME` / `DB_PASSWORD`) | **Production** — persistent, schema migrated automatically on startup. This is the implicit default; no need to set it |
| `in-memory-db` | In-memory | **Quick tests only** — no database needed, but all data is lost on restart. **Not recommended beyond a first look** |

```bash
# Docker — quick test only, not recommended
-e SPRING_PROFILES_ACTIVE=in-memory-db

# Release bundle — quick test only, not recommended
SPRING_PROFILES_ACTIVE=in-memory-db ./scripts/flink-manager.sh
```

> ⚠️ `SPRING_PROFILES_ACTIVE=in-memory-db` is meant for quick tests only: users, configuration,
> history and notifications are wiped on every restart. For anything beyond a first look, use
> the default profile with PostgreSQL.

---

## Next steps

- [Licensing](04-licensing.md)
- [Screenshots](05-screenshots.md)
