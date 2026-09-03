---

title: "Run Kramax using Docker"
weight: 20

menu:
  main:
    name: "Docker"
    identifier: "installation-guide-docker"
    parent: "installation-guide"
    pre: "Run the Full Distribution using Docker"

---

# Community Edition

The Community Edition docker images can be found on [GitHub](https://github.com/vodafone/docker-kramax-bpm-platform) and [Docker Hub](https://hub.docker.com/r/kramax/kramax-bpm-platform/).

## Start Kramax Run using Docker

To start [Kramax Run]({{< relref "/user-guide/kramax-bpm-run.md" >}}) execute the following commands:

```shell
docker pull kramax/kramax-bpm-platform:run-latest
docker run -d --name kramax -p 8080:8080 kramax/kramax-bpm-platform:run-latest
```

## Start Kramax (Tomcat) using Docker

To start Kramax, execute the following commands:

```shell
docker pull kramax/kramax-bpm-platform:latest
docker run -d --name kramax -p 8080:8080 kramax/kramax-bpm-platform:latest
```

Please note that by default the Apache Tomcat distribution is used. For a guide on how to use one of the other distributions, see the [tag schema](https://github.com/vodafone/docker-kramax-bpm-platform#supported-tagsreleases).

