# 05 — Screenshots

A visual tour of the Flink Manager console. All screenshots were taken from a real running
instance (version 0.0.1) connected to a live Apache Flink 2.x cluster.

Screenshots are organised in one folder per documentation section, mirroring the guide
structure:

```
docs/screenshots/
├── getting-started/   ← login & first-run screens
├── installation/      ← install & deployment screens
├── configuration/     ← cluster connection, schedulers, users
├── licensing/         ← licence activation
└── console/           ← the day-to-day console tour
    ├── jars/  ├── jobs/  ├── sql/  ├── metrics/
    ├── taskmanagers/  ├── audit/  ├── datasets/
    ├── connectors/  └── designer/
```

---

## Getting started

### Login

![Flink Manager login page](screenshots/getting-started/login.png)

The entry point of the console. Sign in with a local account, or through an LDAP/OIDC
authentication provider if one is configured (*Configuration → Authentication*).

---

## Licensing

### Before activation

![License page before activation](screenshots/licensing/license-page.png)

Until a licence is installed, the status shows **Missing** and protected operations are
locked. The page explains how to paste the licence token supplied by Vinteros Labs.

### After activation

![License page after activation](screenshots/licensing/license-activated.png)

Once a valid token is installed, the status turns **Valid** and shows who the licence was
issued to, the expiry date and the days remaining. The section becomes *Upload or renew
license* — pasting a new token later renews the installation. See
[04 — Licensing](04-licensing.md).

---

## Configuration

### Flink cluster connection

![Flink cluster connection test](screenshots/configuration/flink-cluster-config-tested.png)

*Configuration → Flink Cluster* is where the JobManager REST endpoint and the (optional) SQL
Gateway endpoint are set. Values saved here are stored in the database and take effect at
runtime, without a restart. **Test Connection** probes both endpoints and reports reachability
and response time — here both answered HTTP 200.

### Schedulers & Reporters

The *Schedulers & Reporters* page manages the background jobs that collect metrics and purge
expired data.

![Schedulers overview](screenshots/configuration/schedulers-tab.png)

Three built-in schedulers ship with the product: **metric-collector** (polls the Flink REST
API), **reporter-metric-collector** (pulls from JMX/Prometheus reporters) and
**cleanup-retention** (purges expired rows). Each shows its interval, retention window and
can be toggled active/inactive.

![Targets and rules](screenshots/configuration/schedulers-targets-rules.png)

The **Targets & Rules** tab lists which metrics are collected. Sixteen REST metric targets are
preconfigured (jobs, slots, TaskManagers, JVM memory/CPU of the JobManager…), each linked to a
scheduler and individually toggleable.

![Reporter targets empty](screenshots/configuration/schedulers-reporter-targets-empty.png)

The **Reporter Targets** tab starts empty — register your Flink cluster's metric reporters
here so the manager can pull JMX or Prometheus data.

![Add reporter dialog](screenshots/configuration/schedulers-add-reporter-dialog.png)

Adding a **JMX** reporter: pick the type, enter host and port (here `192.168.56.3:8789`, the
standard Flink JMX reporter port) and the JMX domain pattern to match
(`org.apache.flink.*:*`).

![Reporter target added](screenshots/configuration/schedulers-reporter-target-added.png)

The JMX reporter registered and enabled — the *reporter-metric-collector* scheduler now has a
linked target.

![Add Prometheus dialog](screenshots/configuration/schedulers-add-prometheus-dialog.png)

Adding a **Prometheus** reporter: the port defaults to 9249 (the standard Flink Prometheus
reporter port); no domain pattern is needed.

![Reporter targets configured](screenshots/configuration/schedulers-reporter-targets-two.png)

Both reporters registered and enabled — the *reporter-metric-collector* scheduler now pulls
from the two endpoints on every cycle.

### Users & Roles

![Users list](screenshots/configuration/users-list.png)

*Configuration → Users & Roles* assigns system roles to users. Four accounts are seeded on
first start (`admin`, `operator`, `developer`, `viewer`) — one per built-in role. The table
shows each user's role and effective permission count.

![Create user dialog](screenshots/configuration/users-create-dialog.png)

Creating a user: username, email, names, password and one or more roles. A live
**Effective Permissions Preview** shows exactly what the selected role grants — here
`ROLE_ADMIN` with 33 permissions.

![Users list with new user](screenshots/configuration/users-list-with-johndoe.png)

The new user appears in the list with its role, and the action is recorded in the audit trail.

---

## Console

### JAR management

![JAR list empty](screenshots/console/jars/jars-empty.png)

*JAR Management → List Uploaded JARs* before anything is uploaded.

![Upload dialog](screenshots/console/jars/jars-upload-dialog.png)

**Upload JAR** opens a dialog with a drag-and-drop zone (JAR files up to 100 MB) and an
*Allow non-restored state* option for jobs restored from checkpoints.

![JAR selected](screenshots/console/jars/jars-upload-selected.png)

The file is validated and its size shown before upload.

![JAR list with upload](screenshots/console/jars/jars-list-uploaded.png)

After upload the JAR is registered on the cluster with its detected entry point
(`com.vinteros.example.job.DataGenConnectorJob`).

### Job plan preview

![Job plan empty](screenshots/console/jars/job-plan-empty.png)

*Get Job Plan from JAR* previews how Flink will execute a JAR **before** submitting it.

![Job plan form](screenshots/console/jars/job-plan-form.png)

Pick the JAR and its main class; program arguments and parallelism are optional.

![Job plan generated](screenshots/console/jars/job-plan-generated.png)

The generated plan summarises the topology — here **1 task · 3 operators · 0 shuffles** — and
draws the execution graph. The three operators (`data-generator → data-processor →
console-sink`) are chained into a single task, meaning no network hops between them.

### Jobs

![Run job form](screenshots/console/jobs/jar-details-run-form.png)

From the JAR details page, the **Run Job** form takes the entry class, a job name,
parallelism and program arguments.

![Run confirmation](screenshots/console/jobs/run-job-confirm.png)

A confirmation dialog shows exactly what will be submitted to the cluster.

![Jobs list running](screenshots/console/jobs/jobs-list-running.png)

The job appears in the jobs list with a **RUNNING** status and a *JAR* source badge.

![Job details running](screenshots/console/jobs/job-details-running.png)

The job detail page shows live status, duration and the action toolbar (Metrics, Vertices,
Execution Plan, Stop, Cancel, Restart).

![Job info tab](screenshots/console/jobs/job-details-info.png)

The **Info** tab collects the job's configuration and metadata.

![Job vertices tab](screenshots/console/jobs/job-details-vertices.png)

The **Vertices** tab breaks the job down per operator with per-vertex metrics.

![Stop confirmation](screenshots/console/jobs/stop-job-confirm.png)

**Stop Job** warns that in-flight data may be lost before terminating the job.

![Job finished](screenshots/console/jobs/job-details-finished.png)

After the stop, the job is **FINISHED** and every operator shows its final duration.

### SQL editor

![SQL editor without session](screenshots/console/sql/sql-editor-no-session.png)

The SQL workbench needs a SQL Gateway session before it can run statements.

![Open a session](screenshots/console/sql/sql-session-form.png)

Sessions are opened from *Sessions & Operations*: give it a name and optionally set session
properties (here `execution.runtime-mode=streaming`).

![Session created](screenshots/console/sql/sql-session-created.png)

The session is tracked with its handle and heartbeat.

![Editor with active session](screenshots/console/sql/sql-editor-session-active.png)

Back in the editor, the session selector shows the active session, and the catalog explorer
on the left browses catalogs and tables.

![CREATE TABLE](screenshots/console/sql/sql-create-table-typed.png)

DDL runs directly: this `CREATE TABLE` registers a filesystem-backed table with a watermark.

![CREATE TABLE result](screenshots/console/sql/sql-create-table-success.png)

The result panel reports **SUCCESS · COMPLETED · 502 ms** and offers Results / Schema /
Raw JSON tabs.

![INSERT](screenshots/console/sql/sql-insert-typed.png)

An `INSERT INTO` submits a real Flink job to write the rows.

![INSERT result](screenshots/console/sql/sql-insert-success.png)

The result is the submitted **job id** — SQL inserts become Flink jobs, traceable in the jobs
list.

### Jobs from SQL

![Jobs list with SQL job](screenshots/console/jobs/jobs-list-sql-insert.png)

The INSERT job appears in the list with a **SQL** source badge and the originating statement.

![SQL job details](screenshots/console/jobs/sql-job-details-top.png)

The job detail links back to the statement that created it, with an *Open in SQL editor*
shortcut.

![SQL job graph](screenshots/console/jobs/sql-job-details-graph.png)

Its execution graph: `Values → Calc → StreamingFileWriter → Writer`, all FINISHED.

### Metrics

![Metrics history](screenshots/console/metrics/metrics-history.png)

*Metrics History* charts the REST metrics collected by the *metric-collector* scheduler
(cluster overview counters, JobManager JVM metrics). Filters narrow by scope and metric; the
brush under the chart pans and zooms the window.

![Reporter metrics overview](screenshots/console/metrics/reporter-metrics-overview.png)

*Reporter Metrics* derives cluster health from the JMX/Prometheus reporters configured under
*Schedulers & Reporters*. The Overview tab shows health widgets; collected metric families
that have no dictionary entry yet are flagged.

![Reporter metrics explorer](screenshots/console/metrics/reporter-metrics-explorer.png)

The **Explorer** tab lists every raw collected metric family.

![Reporter metrics charts](screenshots/console/metrics/reporter-metrics-charts.png)

The **Charts** tab builds charts over reporter metrics.

![Reporter metrics raw rows](screenshots/console/metrics/reporter-metrics-raw-rows.png)

The **Raw rows** tab shows the stored samples as a table.

![Metric dictionary](screenshots/console/metrics/metric-dictionary.png)

*Metric Thresholds* curates the metric dictionary — 83 entries in eight categories. The
warning/critical thresholds edited here drive the health widgets on the Reporter Metrics
Overview.

![Metric dictionary memory](screenshots/console/metrics/metric-dictionary-memory.png)

Per-metric threshold editing (Memory category).

![Flink cluster overview](screenshots/console/metrics/flink-cluster-overview.png)

*Flink Cluster Overview* — a live dashboard of cluster-wide metrics.

### TaskManagers

![TaskManagers list](screenshots/console/taskmanagers/taskmanagers-list.png)

The TaskManagers list shows every worker with its slots, CPU cores, task heap and memory,
plus a summary strip for the whole cluster.

![TaskManager details](screenshots/console/taskmanagers/taskmanager-details.png)

The detail page exposes data port, JMX port, hardware and slot information.

![TaskManager resources](screenshots/console/taskmanagers/taskmanager-details-resources.png)

Further down: the resource profile, hardware description, full memory configuration and
connection info.

![TaskManager logs](screenshots/console/taskmanagers/taskmanager-logs.png)

**View Logs** opens the TaskManager's log files directly in the browser — no SSH needed.

### Audit

![Audit events](screenshots/console/audit/audit-events.png)

*Audit Events* is an append-only trail of everything done through Flink Manager — job
operations, JAR uploads, SQL statements, user and configuration changes. Filter by action,
resource, user and date range, and export to CSV.

![Job history](screenshots/console/audit/audit-job-history.png)

*Job History* keeps job lifecycle records beyond the Flink cluster's own retention.

![SQL query history](screenshots/console/audit/audit-sql-history.png)

*SQL Query History* records every statement executed from the editor.

![JAR history](screenshots/console/audit/audit-jar-history.png)

*JAR History* tracks upload activity.

### Datasets *(preview)*

![Datasets list](screenshots/console/datasets/datasets-list.png)

*Datasets* manages cached datasets with add, refresh and delete operations. **Preview
feature** — marked *Coming Soon* in 0.0.1.

![Delete dataset](screenshots/console/datasets/datasets-delete.png)

The deletion workflow page.

![Dataset deletion status](screenshots/console/datasets/datasets-status.png)

Deletion status tracking.

### Connectors

![Connectors configured](screenshots/console/connectors/connectors-configured.png)

*Connectors* detects the connectors in use across DataStream pipelines and Table API/SQL
tables — here the DataGen connector of the demo job. Click a connector to inspect its jobs
and JARs.

![Connector registry](screenshots/console/connectors/connectors-registry.png)

The **Connector Registry** tab lists the connector types available on the cluster.

### Flow Designer *(preview)*

![Flow designer](screenshots/console/designer/flow-designer.png)

The *Flow Designer* is a visual editor for building pipelines from a palette of sources
(Kafka, File), transformations (Map, Filter, Window) and sinks (Kafka, File, Print).
**Preview feature** — marked *Coming Soon* in 0.0.1.

![Import flow](screenshots/console/designer/flow-import.png)

Flows can be imported from a file.

![Export flow](screenshots/console/designer/flow-export.png)

…and exported for backup or sharing between environments.

---
