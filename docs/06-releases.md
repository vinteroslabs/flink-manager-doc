# 06 — Releases

This page lists the released versions of Flink Manager and their highlights.

## Version table

| Version | Release date | Highlights |
|---|---|---|
| [0.0.2](#002--security-hardening) | 2026-10-27 | Security hardening, multi-platform Docker image |
| [0.0.1](#001--first-release) | 2026-10-17 | First public release |

## Artifacts

| Channel | Location |
|---|---|
| Release bundle (zip) | [GitHub Releases](https://github.com/vinteroslabs/flink-manager-doc/releases) — `flink-manager-<version>.zip` |
| Docker image | [`vinteroslabs/flink-manager:<version>`](https://hub.docker.com/repository/docker/vinteroslabs/flink-manager) on Docker Hub (plus `latest`) |

## Changelog

### 0.0.2 — Security hardening

*2026-10-27*

Second release of Flink Manager. This release closes the vulnerabilities found in the Docker
Hub scan of the `0.0.1` image and ships a truly portable Docker image.

**Security fixes**

- Upgraded the underlying application stack to the currently maintained versions, replacing
  the components that drove the scan findings (web server, framework, security, JSON and
  database driver libraries).
- Removed vulnerable legacy dependencies that were no longer needed.
- Docker base image upgraded to a maintained long-term-support Java runtime, with OS packages
  patched at image build time.
- Internal hardening across the security, user-role and licence code paths, plus new
  integration tests.

**Docker image compatibility**

- Images are now built as **multi-platform manifests (`linux/amd64` + `linux/arm64`)** and
  published under a single tag. The `0.0.1` image was published with only the architecture of
  the build machine; from this release on, the same tag pulls natively on x86-64 hosts
  (Windows / Linux) and on Apple Silicon / ARM servers — no emulation flags needed.

**New features**

- Configurable OpenAPI / Swagger UI deployment settings (UI exposure, path, etc.).

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
