---

title: "Spring Boot Integration"
weight: 55

menu:
  main:
    identifier: "user-guide-spring-boot-integration"
    parent: "user-guide"

---

The KramaX Engine can be used in a Spring Boot application by using provided Spring Boot starters.
Spring boot starters allow to enable behavior of your spring-boot application by adding dependencies to the classpath.

These starters will pre-configure the KramaX process engine, REST API and Web applications, so they can easily be used in a standalone process application.

If you are not familiar with [Spring Boot](http://projects.spring.io/spring-boot/), read the [getting started](http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#getting-started) guide.

To enable KramaX auto configuration, add the following dependency to your ```pom.xml```:

```xml
<dependency>
  <groupId>org.vodafone.kramaX.bpm.springboot</groupId>
  <artifactId>kramaX-bpm-spring-boot-starter</artifactId>
  <version>{{< minor-version >}}.0</version>
</dependency>
```

This will add the KramaX engine v.{{< minor-version >}}.0 to your dependencies.

Other starters that can be used are: 

* [`kramaX-bpm-spring-boot-starter-rest`](rest-api)
* [`kramaX-bpm-spring-boot-starter-webapp`](webapps)
* [`kramaX-bpm-spring-boot-starter-external-task-client`]({{< relref "/user-guide/ext-client/spring-boot-starter.md" >}})

# Requirements

KramaX Spring Boot Starter requires Java 21.

# Supported deployment scenarios

Following deployment scenario is supported by KramaX:

* executable JAR with embedded Tomcat and one embedded process engine (plus Webapps when needed)

There are other possible variations that might also work, but are not tested by KramaX at the moment.
