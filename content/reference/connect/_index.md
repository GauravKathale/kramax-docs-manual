---

title: 'KramaX Connector Reference'
weight: 70
layout: "single"

menu:
  main:
    name: "Connectors"
    identifier: "connect-ref"
    parent: "references"

---

KramaX Connect provides a simple API for connecting HTTP services and other
things. It aims at two usage scenarios: usage in a generic system such as the
KramaX process engine and standalone usage via API.

# Connectors

KramaX Connect provides a HTTP and a SOAP HTTP connector. If you want to
add an own connector to Connect please have a look at the [extending Connect]({{< relref "/reference/connect/extending-connect.md" >}})
section. This section also describes the usage of a `ConnectorConfigurator` to
configure the connector instances.

During the request invocation of a connector an interceptor chain is passed.
The user can add own interceptors to this chain. The interceptor is called for
every request of this connector.

```java
connector.addRequestInterceptor(interceptor).createRequest();
```

# Maven Coordinates

Connect can be used in any Java-based application by adding the following maven
dependency to your `pom.xml` file:

{{< note title="KramaX BOM" class="info" >}}
If you use other KramaX projects please import the
[KramaX BOM]({{< relref "/get-started/apache-maven/" >}})
to ensure correct versions for every KramaX project.
{{< /note >}}

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.vodafone.kramaX.connect</groupId>
      <artifactId>kramaX-connect-bom</artifactId>
      <scope>import</scope>
      <type>pom</type>
      <version>${version.kramaX}</version>
    </dependency>
  </dependencies>
</dependencyManagement>
```

```xml
<dependencies>
  <dependency>
    <groupId>org.vodafone.kramaX.connect</groupId>
    <artifactId>kramaX-connect-core</artifactId>
  </dependency>

  <dependency>
    <groupId>org.vodafone.kramaX.connect</groupId>
    <artifactId>kramaX-connect-connectors-all</artifactId>
  </dependency>
</dependencies>
```

KramaX Connect is published to [maven central](https://mvnrepository.com/artifact/org.vodafone.kramaX.connect).

{{< note title="Process engine plugin" class="info" >}}
If you are using Connect in the KramaX process engine, you also need the `kramaX-engine-plugin-connect` dependency. For more information, refer to the [Connectors guide]({{< relref "/user-guide/process-engine/connectors.md" >}}).
{{< /note >}}

# Logging

KramaX Connect uses [kramaX-commons-logging](https://github.com/vodafone/kramaX-bpm-platform/tree/master/commons/logging) which itself uses [SLF4J](http://slf4j.org) as
a logging backend. To enable logging a SLF4J implementation has to be part of
your classpath. For example `slf4j-simple`, `log4j12` or `logback-classic`.

Apache HTTP Client 5.x has built-in SLF4J support, so no additional bridges are required for HTTP client logging.
