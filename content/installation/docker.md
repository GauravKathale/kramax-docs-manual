---

title: "Run Kramaxx using Docker"
weight: 20

menu:
  main:
    name: "Docker"
    identifier: "installation-guide-docker"
    parent: "installation-guide"
    pre: "Run the Full Distribution using Docker"

---

# Community Edition

The Community Edition docker images can be found on [GitHub](https://github.com/vodafone/docker-kramaxx-bpm-platform) and [Docker Hub](https://hub.docker.com/r/kramaxx/kramaxx-bpm-platform/).

## Start Kramaxx Run using Docker

To start [Kramaxx Run]({{< relref "/user-guide/kramaxx-bpm-run.md" >}}) execute the following commands:

```shell
docker pull kramaxx/kramaxx-bpm-platform:run-latest
docker run -d --name kramaxx -p 8080:8080 kramaxx/kramaxx-bpm-platform:run-latest
```

## Start Kramaxx (Tomcat) using Docker

To start Kramaxx, execute the following commands:

```shell
docker pull kramaxx/kramaxx-bpm-platform:latest
docker run -d --name kramaxx -p 8080:8080 kramaxx/kramaxx-bpm-platform:latest
```

Please note that by default the Apache Tomcat distribution is used. For a guide on how to use one of the other distributions, see the [tag schema](https://github.com/vodafone/docker-kramaxx-bpm-platform#supported-tagsreleases).

