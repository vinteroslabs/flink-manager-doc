# 02 — Installation

> ⚠️ **Licensed software.** You must hold a valid commercial licence from
> [vinteros-labs.com](https://vinteros-labs.com/) before deploying Flink Manager.
> See [LICENSE-INFO.md](../LICENSE-INFO.md).

This guide covers installing Flink Manager for production use. Three supported paths:

- **A. Docker Compose** — recommended; the template below is production-ready.
- **B. Docker image** — plain `docker run` equivalent of the compose template.
- **C. Release bundle** — a zip with the application jar, start scripts and a systemd unit,
  for hosts without Docker.

All paths run the same application: the web console is served by the application itself at
`/flink-manager/`, so there is nothing else to deploy.

## Prerequisites

- A **PostgreSQL** database (14+). Create the database before first start — Flink Manager
  creates and migrates the schema automatically.
- A reachable **Apache Flink 2.x** cluster (JobManager REST API), and optionally a
  **Flink SQL Gateway**.
- Secrets to set: a strong `JWT_SECRET`, an `AUTH_SECRET_KEY` and a database password.

> For a quick evaluation **without** a database, see
> [01 — Getting started](01-getting-started.md#2-quick-test-without-a-database) (in-memory
> profile; data is lost on restart).

---

## A. Docker Compose

Create a `.env` file next to your `docker-compose.yml`:

```dotenv
# .env — the compose file refuses to start without these
DB_PASSWORD=<db-password>
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

For production, set in `.env`:

```dotenv
SPRING_PROFILES_ACTIVE=default
DB_URL=jdbc:postgresql://db-prod:5432/flink-manager
DB_USERNAME=flinkmgr
```

Then:

```bash
docker compose up -d
curl http://localhost:8080/actuator/health
```

> 💡 The template pins `image: latest` with `pull_policy: always`. To stay on a fixed version,
> replace `latest` with the release tag (e.g. `0.0.2`) — see [06 — Releases](06-releases.md).

### Upgrades

```bash
docker compose pull && docker compose up -d
```

The database schema is migrated automatically on startup; no manual migration steps.

---

## B. Docker image (plain `docker run`)

The image is published on Docker Hub as
[`vinteroslabs/flink-manager`](https://hub.docker.com/repository/docker/vinteroslabs/flink-manager),
tagged per release (`0.0.2`) plus a rolling `latest`.

```bash
docker run -d --name flink-manager \
  --restart unless-stopped \
  -p 8080:8080 \
  -e SPRING_PROFILES_ACTIVE=default \
  -e DB_URL="jdbc:postgresql://<db-host>:5432/flink-manager" \
  -e DB_USERNAME="flinkmgr" \
  -e DB_PASSWORD="<db-password>" \
  -e FLINK_SERVER_URL="http://<flink-jobmanager>:8081" \
  -e FLINK_SQL_GATEWAY_URL="http://<flink-sqlgateway>:8083" \
  -e JWT_SECRET="<random-base64-secret>" \
  -e AUTH_SECRET_KEY="<random-32-bytes-base64>" \
  vinteroslabs/flink-manager:0.0.2
```

Notes:

- The container listens on port **8080** (override with `SERVER_PORT`).
- Memory defaults are `-Xms512m -Xmx1g` with G1 GC; tune with `JVM_OPTS`.
- All settings are environment variables — see
  [03 — Configuration](03-configuration.md) for the full list.

### Verify

```bash
curl http://localhost:8080/actuator/health
```

Then open `http://<host>:8080/flink-manager/` and log in (see
[Getting started → Log in](01-getting-started.md#3-log-in)).

### Upgrades

Pull the new tag, stop and recreate the container with the same environment. Schema migrations
run automatically on startup.

---

## C. Release bundle

Each release publishes a zip on
[GitHub Releases](https://github.com/vinteroslabs/flink-manager-doc/releases):

```
flink-manager-<version>.zip
├── flink-manager-api-<version>-protected.jar
└── scripts/
    ├── flink-manager.sh        # Linux/macOS launcher (foreground | start | stop)
    ├── flink-manager.bat       # Windows launcher
    └── flink-manager.service   # systemd unit
```

### 1. Prerequisites

- **Java 25+** (JDK) on the host
- A PostgreSQL database as above

### 2. Install

```bash
sudo mkdir -p /opt/flink-manager
sudo unzip flink-manager-0.0.2.zip -d /opt/flink-manager
sudo useradd --system --home /opt/flink-manager flinkmgr
sudo chown -R flinkmgr:flinkmgr /opt/flink-manager
```

### 3. Configure

Every setting can be provided as an environment variable (see
[03 — Configuration](03-configuration.md)). With systemd, override without editing the unit:

```bash
sudo systemctl edit flink-manager
```

```ini
[Service]
Environment=SPRING_PROFILES_ACTIVE=default
Environment=DB_URL=jdbc:postgresql://db-prod:5432/flink-manager
Environment=DB_USERNAME=flinkmgr
Environment=DB_PASSWORD=<secret>
Environment=FLINK_SERVER_URL=http://flink-prod:8081
Environment=JWT_SECRET=<base64-secret>
Environment=AUTH_SECRET_KEY=<base64-secret>
# Secrets can also live in a root-only file referenced with EnvironmentFile=.
```

### 4. Run with systemd

```bash
sudo install -m 644 /opt/flink-manager/scripts/flink-manager.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now flink-manager
journalctl -u flink-manager -f
```

The unit runs the jar as the unprivileged `flinkmgr` user and restarts it on failure.

### 5. Run without systemd

```bash
cd /opt/flink-manager
./scripts/flink-manager.sh start     # background (writes flink-manager.pid, logs in logs/)
./scripts/flink-manager.sh stop      # stop
./scripts/flink-manager.sh           # foreground
```

On Windows, run `scripts\flink-manager.bat` (foreground).

#### Calling the launcher from your own script

Every setting in `flink-manager.sh` can be overridden via the environment before invoking it.
The recommended pattern is a small wrapper script (`start.sh`) that sets the parameters your
deployment needs, then calls the launcher:

```bash
#!/usr/bin/env bash
# start.sh — configure Flink Manager, then start it.

# Set strong, private secrets before running start.sh.
JWT_SECRET=CBEuoEuoKPjU66ncgAThTdUsa+xGXECTh+pqUClEx/JFWDCTR9cFH7WJegyvI7xW
AUTH_SECRET_KEY=NhMXw1GEQpK25NgY2YDWBk8kanFMnmm3s3S/LNvz+wM=

# The release launcher defaults to localhost for Flink endpoints.
FLINK_SERVER_URL=http://localhost:8081
FLINK_SQL_GATEWAY_URL=http://localhost:8083
FLINK_SAVEPOINT_DIR=/tmp/flink-savepoints
JWT_EXPIRATION_MS=43200000
AUTH_UI_BASE_URL=http://localhost:8080/flink-manager
# CORS_ALLOWED_ORIGINS is intentionally not set: the console is served by the
# application itself, so no cross-origin permission is needed.
SERVER_PORT=8080
SPRING_PROFILES_ACTIVE=in-memory-db

exec /opt/flink-manager/scripts/flink-manager.sh "$@"
```

```bash
chmod +x start.sh
./start.sh              # foreground
./start.sh start        # background (delegates to flink-manager.sh start)
./start.sh stop         # stop
```

Notes:

- **Secrets are the only mandatory overrides.** `JWT_SECRET` and `AUTH_SECRET_KEY` ship with
  dev-only defaults and must be replaced; everything else falls back to a sensible default
  (see [03 — Configuration](03-configuration.md)).
- The example above uses `SPRING_PROFILES_ACTIVE=in-memory-db` for a quick test with no
  database. For production, use the default profile with PostgreSQL — see
  [Spring profiles](03-configuration.md#spring-profiles).
- Keep secrets out of the wrapper if the script lives in version control: source them from a
  root-only file instead (`source /etc/flink-manager/secrets.env`), or rely on the systemd
  `EnvironmentFile=` mechanism (see step 4).
- Passing `"$@"` through means `start`/`stop` still work: `./start.sh start`, `./start.sh stop`.

### 6. Verify

```bash
curl http://localhost:8080/actuator/health
```

### 7. Upgrades

Install the new jar **and** the matching `flink-manager.sh` from the new bundle (the script is
stamped with the version), remove the old jar, then restart the service. Schema migrations run
automatically on startup.

---

## Reverse proxy & TLS

Flink Manager can terminate TLS itself (set `SSL_ENABLED=true` and point `SSL_KEYSTORE_PATH`
at a PKCS12/JKS keystore — see [03 — Configuration](03-configuration.md)), or you can put it
behind a reverse proxy that forwards `/` to the application port.

The console is served by the application itself, so when the proxy routes everything to the
same origin the browser makes same-origin API calls and **no CORS configuration is needed**.
Set `CORS_ALLOWED_ORIGINS` only if the console is reachable under a *different* origin than
the API (e.g. the proxy serves the console on `https://flink.example.com` but forwards API
calls to a different host).

---

## Licence activation

After installation, activate your licence as described in [04 — Licensing](04-licensing.md).

---

## Next steps

- [Configuration reference](03-configuration.md)
- [Licensing](04-licensing.md)
