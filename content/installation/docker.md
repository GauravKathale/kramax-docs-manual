---

title: "Run KramaX using Docker"
weight: 20

menu:
  main:
    name: "Docker"
    identifier: "installation-guide-docker"
    parent: "installation-guide"
    pre: "Run the Full Distribution using Docker"

---

# Community Edition

The Community Edition docker images can be found on [GitHub](https://github.com/vodafone/docker-kramaX-bpm-platform) and [Docker Hub](https://hub.docker.com/r/kramaX/kramaX-bpm-platform/).

## Start KramaX Run using Docker

To start [KramaX Run]({{< relref "/user-guide/kramaX-bpm-run.md" >}}) execute the following commands:

```shell
docker pull kramaX/kramaX-bpm-platform:run-latest
docker run -d --name kramaX -p 8080:8080 kramaX/kramaX-bpm-platform:run-latest
```

## Start KramaX (Tomcat) using Docker

To start KramaX, execute the following commands:

```shell
docker pull kramaX/kramaX-bpm-platform:latest
docker run -d --name kramaX -p 8080:8080 kramaX/kramaX-bpm-platform:latest
```

Please note that by default the Apache Tomcat distribution is used. For a guide on how to use one of the other distributions, see the [tag schema](https://github.com/vodafone/docker-kramaX-bpm-platform#supported-tagsreleases).

