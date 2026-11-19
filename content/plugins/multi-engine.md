---

title: "Multi-Engine Integration and Configuration Guide"
weight: 10

menu:
  main:
    name: "Multi-Engine"
    identifier: "plugins-multi-engine"
    parent: "plugins"
    pre: "Run several process engines in one Kramax application"

---

Multi-engine lets one Kramax application run several process engines, each with its own database. This guide explains what it does, how to integrate it into a Kramax Spring Boot application, and what every configuration field means.

| Component | Details |
|-----------|---------|
| Artifact | `org.vodafone.kramax:kramax-integrated-monitoring:0.0.2-SNAPSHOT` |
| Platform | Kramax 3.1.0, Spring Boot 4.1.2 |
| Java | 21 or later |
| Date | 5 November 2026 |

# Overview

| | |
|---|---|
| What it does | Adds process engines next to the default engine, all managed by one application. |
| Problem it solves | Business areas stay separated without running a separate application for each one. |
| What you add | One Maven dependency, plus a database and a few properties per engine. |
| Code changes | None to add an engine. Deploying processes to an engine uses the Kramax API. |

## What your organisation gets

* **Separated workloads.** Each engine has its own database, deployments, permissions and history, so one business area cannot see another's data through the engine.
* **One application to run.** All engines appear in the Kramax webapps and REST API, so there is one thing to deploy, monitor and upgrade.
* **Background work included.** One shared job executor runs timers, async steps and retries for every engine.
* **Same rules everywhere.** Your Kramax settings, including permission checks, apply to every engine unless you override them.

## How it fits in

{{< img src="../multi-engine-diagram.png" title="Figure 1: Several engines inside one Kramax application" >}}

# Before You Start

## Requirements

| Item | Requirement |
|------|-------------|
| Kramax | 3.1.0 with `kramax-bpm-spring-boot-starter-webapp` and `kramax-bpm-spring-boot-starter-rest` |
| Spring Boot | 4.1.2 |
| Java | Java 21 or later |
| Databases | One database per additional engine, reachable from the application |

## Who does what

| Role | Responsibility |
|------|----------------|
| Application developer | Adds the dependency and properties, builds the application and deploys processes |
| Database administrator | Creates a database and account for each additional engine |
| Operations | Stores the database passwords, sizes the thread and connection pools, and runs the deployment |
| Kramax administrator | Checks access after go-live and manages permissions on each engine in the Admin webapp |

## Information to collect

For each additional engine: a name, JDBC URL, username and password.

# Integration Steps

## Step 1: Prepare the databases

Create an empty database and an account for each engine. Do not share one database between two engines. With schema update enabled, Kramax creates the tables on first start.

## Step 2: Add the dependency

```xml
<dependency>
  <groupId>org.vodafone.kramax</groupId>
  <artifactId>kramax-integrated-monitoring</artifactId>
  <version>1.0.0-SNAPSHOT</version>
</dependency>
```

## Step 3: Declare the engines

```properties
kramax.multi-engine.enabled=true

kramax.engines.order.url=jdbc:postgresql://<order-db-host>:5432/order
kramax.engines.order.username=${ORDER_DB_USER}
kramax.engines.order.password=${ORDER_DB_PASSWORD}
kramax.engines.order.driver-class-name=org.postgresql.Driver

kramax.engines.inventory.url=jdbc:postgresql://<inventory-db-host>:5432/inventory
kramax.engines.inventory.username=${INVENTORY_DB_USER}
kramax.engines.inventory.password=${INVENTORY_DB_PASSWORD}
kramax.engines.inventory.driver-class-name=org.postgresql.Driver
```

The engine name is what follows `kramax.engines.`. It appears in the engine selector of the webapps and in REST URLs such as `/engine-rest/engine/order/process-definition`.

## Step 4: Deploy processes to an engine

Classpath resources are deployed to the default engine only. An additional engine runs only what is deployed to it, so nothing appears there by accident. Deploy from your application code, naming the engine:

```java
ProcessEngines.getProcessEngine("order")
    .getRepositoryService()
    .createDeployment()
    .addClasspathResource("order-process.bpmn")
    .enableDuplicateFiltering(true)
    .deploy();
```

{{< note title="Timing" class="info" >}}
Run this after the application has started, for example in an `ApplicationReadyEvent` listener, because additional engines start after your Spring beans are created. The REST endpoint `/engine-rest/engine/<name>/deployment/create` also works.
{{< /note >}}

## Step 5: Check the engines

Start the application and look for `Started 2 additional process engine(s)` in the log, with your engine names. Then open the webapps and check each engine appears in the engine selector. See [Acceptance Checks](#acceptance-checks) for the full list.

# How Engines Are Set Up

## What is separate and what is shared

| Separate for each engine | Shared by all engines |
|--------------------------|-----------------------|
| Database and connection pool | The application, webapps and REST API |
| Deployed processes and running instances | Sign-in and users |
| Tasks and history | The job executor and its threads |
| Permissions (authorizations) | The `kramax.bpm.*` settings and engine plugins |
| Transactions | Spring beans used in process expressions |

## What each engine gets from the application

| Setting | On additional engines |
|---------|-----------------------|
| Authorization, history, metrics, failed job handling | Same as the application (`kramax.bpm.*`) |
| Generic properties and engine plugins | Same as the application |
| Jobs and timers | Run by the shared job executor. Off for all engines if `kramax.bpm.job-execution.enabled=false` |
| `kramax.bpm.database.*` | Not applied. Each engine uses its own `kramax.engines.<name>.*` |
| Auto deployment | Not applied. Deploy to each engine yourself |

## Giving one engine different settings

Use the `properties` fields of the engine. These override the application:

```properties
kramax.engines.inventory.properties.history=audit
kramax.engines.inventory.properties.default-number-of-retries=5
```

## Using an engine from code

* Look an engine up with `ProcessEngines.getProcessEngine("order")`, or use the Spring bean `orderProcessEngine` once the application has started.
* Injecting `ProcessEngine` still gives the default engine.
* Process expressions can use your Spring beans on every engine.
* `MultiEngineConfiguration` has `createEngine` and `removeEngine` to add and remove engines at runtime.

# Security and Access

## Permissions on each engine

Set `kramax.bpm.authorization.enabled=true`. Permission checks then apply on every engine, so selecting a different engine never gives a user more access.

{{< note title="Permissions are per engine" class="warning" >}}
Permissions are stored per engine. A grant made in the Admin webapp on one engine does not apply to the others, so repeat it on each engine that needs it. Without a grant, a user has no access to that engine.
{{< /note >}}

## With the Azure SSO plugin

Add both dependencies. No extra configuration is needed.

* The `kramax-admin` and `kramax-readonly` grants are created in every engine at startup.
* A user's groups are the same on every engine. Only the permissions behind them differ.

See the Kramax Azure SSO Plugin guide for sign-in and access levels.

# Field Reference

Fields with no default are optional unless marked required.

## Global settings

| Field | Default | What it does |
|-------|---------|--------------|
| `kramax.multi-engine.enabled` | `false` | Turns on the additional engines |
| `kramax.multi-engine.database-schema-update` | `kramax.bpm.database.schema-update` | Default schema update mode for all additional engines |

## Per engine

Prefix: `kramax.engines.<name>.`

| Field | Default | What it does |
|-------|---------|--------------|
| `url` or `jdbc-url` | Required | JDBC URL. If both are set, `jdbc-url` is used |
| `username` | | Database user |
| `password` | | Database password. Keep it in a secret store |
| `driver-class-name` | Detected from the URL | JDBC driver class |
| `pool-name` | `kramax-<name>` | Connection pool name, shown in logs |
| `maximum-pool-size` | Hikari default (10) | Maximum connections for this engine |
| `minimum-idle` | Hikari default | Minimum idle connections |
| `database-schema-update` | See global settings | `true`, `false`, `create`, `create-drop` or `drop-create`. Any other value stops the application from starting |
| `properties.<field>` | | Any engine configuration field for this engine, overriding the application. For example `properties.history=audit` |

Engine names may contain letters, digits, `_` and `-`. The name `default` is reserved.

## Related Kramax settings

| Field | Default | What it does |
|-------|---------|--------------|
| `kramax.bpm.authorization.enabled` | `false` | Turns on permission checks on every engine. Set it to `true` in production |
| `kramax.bpm.job-execution.enabled` | `true` | Turns job execution on or off for all engines |
| `kramax.bpm.job-execution.core-pool-size` | `3` | Job threads kept ready, shared by all engines |
| `kramax.bpm.job-execution.max-pool-size` | `10` | Maximum job threads, shared by all engines |
| `kramax.bpm.job-execution.queue-capacity` | `3` | Jobs that can wait for a free thread |
| `kramax.bpm.history-level` | `full` | History level of all engines. It cannot change for an existing database |

# Complete Example

A production-style configuration with two additional engines. All values in `${...}` come from the environment.

```properties
# ---- Default engine ----
spring.datasource.url=${DEFAULT_DB_URL}
spring.datasource.username=${DEFAULT_DB_USER}
spring.datasource.password=${DEFAULT_DB_PASSWORD}
kramax.bpm.authorization.enabled=true

# ---- Job threads, shared by all engines ----
kramax.bpm.job-execution.core-pool-size=6
kramax.bpm.job-execution.max-pool-size=20

# ---- Additional engines ----
kramax.multi-engine.enabled=true
kramax.multi-engine.database-schema-update=false

kramax.engines.order.url=${ORDER_DB_URL}
kramax.engines.order.username=${ORDER_DB_USER}
kramax.engines.order.password=${ORDER_DB_PASSWORD}
kramax.engines.order.driver-class-name=org.postgresql.Driver
kramax.engines.order.maximum-pool-size=20

kramax.engines.inventory.url=${INVENTORY_DB_URL}
kramax.engines.inventory.username=${INVENTORY_DB_USER}
kramax.engines.inventory.password=${INVENTORY_DB_PASSWORD}
kramax.engines.inventory.driver-class-name=org.postgresql.Driver
```

{{< note title="Schema update" class="info" >}}
`database-schema-update=false` needs the schema to exist already. For a brand-new database, start once with `true`, or create the tables with the Kramax SQL scripts.
{{< /note >}}

# Acceptance Checks

Run these before go-live.

| # | Check | Expected result |
|---|-------|-----------------|
| 1 | Check the startup log | `Started 2 additional process engine(s)` with the expected names |
| 2 | Open the webapps | Every engine appears in the engine selector |
| 3 | Deploy a process to an additional engine | It appears on that engine only |
| 4 | Run a process with a timer or async step on an additional engine | The job runs and the process continues |
| 5 | Run a process that calls one of your Spring beans | The bean is called |
| 6 | As a user with no grant on an engine, select that engine | No data is visible |
| 7 | As a user with limited grants, try a change you have no grant for | Refused with 403, on every engine |

# Going to Production

## Checklist

* `kramax.bpm.authorization.enabled=true` is set.
* Each engine has its own database.
* Database passwords come from a secret store or environment variables.
* Schema changes are controlled, with `database-schema-update=false`.
* Job threads and connection pools are sized for all engines together.
* Permissions are granted on each engine that needs them.
* All acceptance checks pass.

## Sizing

* **Job threads.** All engines share one pool, so size `kramax.bpm.job-execution.*` for their combined load. The default maximum is 10 threads for all engines together, so test it under your expected load.
* **Database connections.** Each engine has its own pool of up to 10 connections by default. Check that the total fits your database limits, across all application instances.

## Operating it

* The history level of an existing database cannot change. An engine fails to start if its level differs from the one in the database.
* Upgrade the schema of every engine when you upgrade Kramax, using the Kramax SQL scripts.
* Set engine names in a properties or YAML file. Environment variables lose upper-case letters, which would rename an engine such as `Inventory`.

## Known limitations

| Limitation | What to do |
|------------|------------|
| A transaction cannot span two engines | Design processes so each one completes within its engine, and use messages between engines |
| All engines must start together | If an engine's database is unreachable at startup, the application does not start. This is deliberate, so a missing engine is never hidden |
| Health checks do not cover additional engines individually | Monitor each database separately |
| Engines added with `createEngine` are not kept | Add them to the properties so they return after a restart |

## Upgrading from an earlier version

If you already run additional engines, read this before upgrading. Earlier versions did not enforce permissions or run jobs on additional engines. Both now work, so behaviour changes on first start.

{{< note title="Waiting jobs will run" class="warning" >}}
Overdue timers and background steps on additional engines will start as soon as the upgrade is live, including any that send emails or call other systems. Review them first:

```sql
SELECT type_, count(*) FROM act_ru_job
WHERE retries_ > 0 AND (duedate_ IS NULL OR duedate_ <= now())
GROUP BY type_;
```
{{< /note >}}

* **Permissions now apply.** Users without a grant on an additional engine lose access there. Add the grants they need, per engine, before you upgrade.
* **Sign in again.** Everyone must sign out and back in after the upgrade.
* **Schema update setting.** `kramax.multi-engine.database-schema-update` is now read, so check that its value is what you want.
* **History level.** Additional engines now follow `kramax.bpm.history-level`, and used `full` before. If you set a different level, also set `kramax.engines.<name>.properties.history=full` for existing engines, or they will not start.
* An engine with no URL now stops startup, where it was silently skipped before.

# Troubleshooting

| What you see | Likely cause | Fix |
|--------------|--------------|-----|
| Engine missing from the selector | Multi-engine not enabled | Set `kramax.multi-engine.enabled=true` and check the startup log |
| `A datasource is required for engine` | Engine has no URL | Set `kramax.engines.<name>.url` |
| `Invalid engine name` | The name has unsupported characters, or is `default` | Use letters, digits, `_` and `-` |
| `Could not start process engine` | Database unreachable, wrong credentials or missing schema | Check the cause in the log, then the connection settings and `database-schema-update` |
| Application fails with a history level error | Configured level differs from the database | Set `kramax.engines.<name>.properties.history` to the level in the database |
| Engine renamed or not found when set by environment variable | Environment variables lose upper-case letters | Use lower-case engine names, or set them in a properties or YAML file |
| Jobs are not running | Job execution disabled | Check `kramax.bpm.job-execution.enabled` |
| Jobs run slowly | One thread pool serves all engines | Raise `kramax.bpm.job-execution.max-pool-size` |
| Process missing on an additional engine | It was deployed to another engine | Deploy it to that engine |
| User has access on one engine but not another | Permissions are stored per engine | Grant them on that engine in the Admin webapp |
| Every user can do everything | Authorization is off | Set `kramax.bpm.authorization.enabled=true` |

# Frequently Asked Questions

**Is data separated between engines?**
Yes. Each engine has its own database, and a query on one engine only sees that engine's data. Users with access to several engines can of course see each of them, within their permissions.

**Can a user have different rights on different engines?**
Yes. Permissions are stored per engine, so the same group can have different grants on each one.

**Can engines use different database types?**
Yes. Each engine has its own connection settings, and the type is detected from the connection. It must be a database Kramax supports.

**Can we add an engine without changing code?**
Yes. Add its properties, then restart the application. You then deploy processes to it.

**How many engines can we run?**
There is no fixed limit. Each engine adds a connection pool and shares the same job threads, so plan for your database limits and load.

**What happens if an engine's database goes down while running?**
Requests and jobs for that engine fail until the database returns. Other engines keep working. The application must be able to reach every engine's database when it starts.

**Can a process on one engine call a process on another?**
Not directly. Engines are separate, so use messages or your own integration between them.

**Does this work with single sign-on?**
Yes. The Kramax Azure SSO plugin works with multi-engine with no extra configuration.
