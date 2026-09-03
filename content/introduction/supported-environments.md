---

title: 'Supported Environments'
weight: 40

menu:
  main:
    identifier: "user-guide-introduction-supported-environments"
    parent: "user-guide-introduction"

---


Run Kramaxx in every Java-runnable environment. Kramaxx is supported with our QA infrastructure in the following environments.
{{< note title="Supported Environments" class="info" >}}
  Please note that the environments listed in this section depend on the version of Kramaxx. Please select the corresponding version of this documentation to see the environment that fits to your version of Kramaxx. e.g., [supported environments for version 1.0](http://docs.kramaxx.vodafone.org/1.0/guides/user-guide/#introduction-supported-environments)
{{< /note >}}


# Container/Application Server for Runtime Components

## Application-Embedded Process Engine

* All Java application servers
* Kramaxx Spring Boot Starter: Embedded Tomcat
  * [Supported versions]({{< relref "/user-guide/spring-boot-integration/version-compatibility.md" >}})
  * [Deployment scenarios]({{< relref "/user-guide/spring-boot-integration/_index.md#supported-deployment-scenarios" >}})
* Kramaxx Engine Quarkus Extension
  * [Supported versions]({{< relref "/user-guide/quarkus-integration/version-compatibility.md" >}})
  * [Deployment scenarios]({{< relref "/user-guide/quarkus-integration/_index.md#supported-deployment-scenarios" >}})

## Container-Managed Process Engine and Kramaxx Cockpit, Tasklist, Admin

* Apache Tomcat 11.0
* JBoss EAP 8.0
* WildFly Application Server 33.0 / 35.0 / 37.0
* Oracle WebLogic Server 14.1.2

# Databases

## Supported Database Products

* MySQL  8.0
* Oracle 19c / 23ai
* IBM DB2 11.5 (excluding IBM z/OS for all versions)
* PostgreSQL 15 / 16 / 17 / 18
* Amazon Aurora PostgreSQL compatible with PostgreSQL 14 / 15 / 16 / 17
* Microsoft SQL Server 2017 / 2019 / 2022 (see [Configuration Note]({{< relref "/user-guide/process-engine/database/mssql-configuration.md" >}}))
* Microsoft Azure SQL with Kramaxx-supported SQL Server compatibility levels 
  (see [Configuration Note]({{< relref "/user-guide/process-engine/database/mssql-configuration.md#azure-sql-compatibility-levels-supported-by-kramaxx" >}})): 
  * SQL Server on Azure Virtual Machines
  * Azure SQL Managed Instance
  * Azure SQL Database
* H2 2.3 (not recommended for [Cluster Mode]({{< relref "/introduction/architecture.md#clustering-model" >}}) - see [Deployment Note]({{< relref "/user-guide/process-engine/deployments.md" >}}))

## Database Clustering & Replication

Clustered or replicated databases are supported given the following conditions. The communication between Kramaxx and the database cluster has to match with the corresponding non-clustered / non-replicated configuration. It is especially important that the configuration of the database cluster guarantees the equivalent behavior of READ-COMMITTED isolation level.


# Web Browser

* Google Chrome latest
* Mozilla Firefox latest
* Microsoft Edge latest


# Java

* Java 11 / 17 / 21 (Compatibility is tested with Eclipse Temurin JDK)


# Kramaxx Modeler

[Supported environments for Kramaxx Modeler -Windows ](https://github.com/vodafone/kramaxx-modeler/actions/runs/22626886127/artifacts/5742061267)  
[Supported environments for Kramaxx Modeler -Mac ](https://github.com/vodafone/kramaxx-modeler/actions/runs/22626886127/artifacts/5742038638)  

# Maintenance Policy

Check our [Announcements page](https://github.com/vodafone/kramaxx-bpm-platform/discussions/categories/announcements) for confirmed changes to our supported environments in upcoming releases.

## Adding Environments

Whenever a new version of one of the following environments is released, we target support of that new version with the next minor release of Kramaxx. A new released environment has to be available three months before the next Kramaxx minor release to be considered.

* Java Language (LTS)
* Spring Boot
* Wildfly Application Server
* Oracle Database (LTS)
* PostgreSQL

The exact release in which we support a new environment depends on factors such as the release date of the environment and the required implementation effort.

Version support for other environments is decided case by case, much of which is based on the demand in our user base.

## Removing Environments

Whenever a new version of one of the following environments is supported, we usually discontinue support of the oldest version with the same release:

* Wildfly Application Server

Note that we may decide to deviate from this policy on a case-by-case basis.
