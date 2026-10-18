# 06 — Releases

This page lists the released versions of Flink Manager and their highlights.

## Version table

| Version | Release date | Highlights |
|---|---|---|
| [0.0.1](#001--first-release) | 2026-10-17 | First public release |

## Artifacts

| Channel | Location |
|---|---|
| Release bundle (zip) | [GitHub Releases](https://github.com/vinteroslabs/flink-manager-doc/releases) — `flink-manager-<version>.zip` |
| Docker image | [`vinteroslabs/flink-manager:<version>`](https://hub.docker.com/repository/docker/vinteroslabs/flink-manager) on Docker Hub (plus `latest`) |

## Changelog

### 0.0.1 — First release

*2026-10-17*

Initial public release of the Flink Manager console.

**Cluster & jobs**

- Flink cluster overview and TaskManager views, including TaskManager logs
- Job run / stop / restart with savepoint support (configurable savepoint directory and
  drain/format defaults)
- JAR management: upload, list, delete, and job-plan preview before running
- Shared jobs snapshot with configurable refresh interval

**SQL**

- SQL workbench backed by the Flink SQL Gateway (optional — disabled until a gateway endpoint
  is configured)
- Saved queries, session & operation history, query history

**Metrics & alerting**

- Metrics history and metric dictionary
- Metric thresholds and reporter metrics
- Schedulers & reporters configuration

**Flows & datasets**

- Visual flow designer with import/export
- Dataset management with deletion workflow

**Governance & operations**

- Users & roles (RBAC) with seeded roles: admin, operator, developer, viewer
- Authentication providers: local accounts, LDAP, OIDC (provider secrets encrypted at rest)
- Audit events plus job / JAR / SQL query history
- Persistent notifications for job and system events
- Product licence management (install / renew via the console, expiry warnings)
- Inbound HTTPS with optional mutual TLS
- Manager settings and cluster shutdown controls

**Deployment**

- Docker image on Docker Hub and release bundle (jar + start scripts + systemd unit)
- PostgreSQL persistence with automatic schema migration on startup

---

## Versioning & publication

- Releases are tagged `v<version>` (e.g. `v0.0.1`) in this repository.
- Every release publishes a zip bundle on
  [GitHub Releases](https://github.com/vinteroslabs/flink-manager-doc/releases) and a
  corresponding Docker tag on
  [Docker Hub](https://hub.docker.com/repository/docker/vinteroslabs/flink-manager).
- The bundle's jar and launcher scripts are version-stamped and must always be installed
  together (see [02 — Installation](02-installation.md#7-upgrades)).
- Release announcements and the full changelog are also published on the official website:
  [vinteros-labs.com/resources/changelog/](https://vinteros-labs.com/resources/changelog/).

## Resources & support

| Resource | Link |
|---|---|
| Online documentation | [vinteros-labs.com/resources/docs/](https://vinteros-labs.com/resources/docs/) |
| Changelog | [vinteros-labs.com/resources/changelog/](https://vinteros-labs.com/resources/changelog/) |
| Blog (release notes, guides, news) | [vinteros-labs.com/resources/blog/](https://vinteros-labs.com/resources/blog/) |

### Reporting issues

If you run into a problem:

1. **Open a GitHub issue** in this repository:
   [github.com/vinteroslabs/flink-manager-doc/issues](https://github.com/vinteroslabs/flink-manager-doc/issues) —
   include the Flink Manager version, how it is deployed (Docker or release bundle), and the
   relevant log excerpts.
2. **Create a ticket on the official website**:
   [vinteros-labs.com](https://vinteros-labs.com/) — preferred for licence, billing and
   support-contract matters, and for anything confidential.
