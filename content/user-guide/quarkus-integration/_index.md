---

title: "Quarkus Integration"
weight: 56

menu:
  main:
    identifier: "user-guide-quarkus-integration"
    parent: "user-guide"

---
{{< deprecated >}}

The KramaX Engine can be used in a Quarkus application by using the provided Quarkus Extension. Quarkus Extensions add 
behavior to your Quarkus application by adding dependencies to the classpath.

The KramaX Engine Quarkus Extension will pre-configure the KramaX process engine, so it can be easily used in a 
Quarkus application.

If you are not familiar with [Quarkus](https://quarkus.io/), have a look at the [getting started](https://quarkus.io/get-started/) guide.

To enable KramaX Engine autoconfiguration, add the following dependency to your `pom.xml`:

```xml
<dependency>
  <groupId>org.vodafone.kramaX.bpm.quarkus</groupId>
  <artifactId>kramaX-bpm-quarkus-engine</artifactId>
  <version>{{< minor-version >}}.0</version>
</dependency>
```

This will add the KramaX engine v.{{< minor-version >}}.0 to your dependencies.

# Supported deployment scenarios

KramaX supports the following deployment scenario:

* executable JAR with one embedded process engine.

There are other possible variations that might also work, but are not tested by KramaX at the moment.
