---

title: 'Kramax Connector Reference'
weight: 70
layout: "single"

menu:
  main:
    name: "Connectors"
    identifier: "connect-ref"
    parent: "references"

---

Kramax Connect provides a simple API for connecting HTTP services and other
things. It aims at two usage scenarios: usage in a generic system such as the
Kramax process engine and standalone usage via API.

# Connectors

Kramax Connect provides a HTTP and a SOAP HTTP connector. If you want to
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

{{< note title="Kramax BOM" class="info" >}}
If you use other Kramax projects please import the
[Kramax BOM]({{< relref "/get-started/apache-maven/" >}})
to ensure correct versions for every Kramax project.
{{< /note >}}

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.vodafone.kramax.connect</groupId>
      <artifactId>kramax-connect-bom</artifactId>
      <scope>import</scope>
      <type>pom</type>
      <version>${version.kramax}</version>
    </dependency>
  </dependencies>
</dependencyManagement>
```

```xml
<dependencies>
  <dependency>
    <groupId>org.vodafone.kramax.connect</groupId>
    <artifactId>kramax-connect-core</artifactId>
  </dependency>

  <dependency>
    <groupId>org.vodafone.kramax.connect</groupId>
    <artifactId>kramax-connect-connectors-all</artifactId>
  </dependency>
</dependencies>
```

Kramax Connect is published to [maven central](https://mvnrepository.com/artifact/org.vodafone.kramax.connect).

{{< note title="Process engine plugin" class="info" >}}
If you are using Connect in the Kramax process engine, you also need the `kramax-engine-plugin-connect` dependency. For more information, refer to the [Connectors guide]({{< relref "/user-guide/process-engine/connectors.md" >}}).
{{< /note >}}

# Logging

Kramax Connect uses [kramax-commons-logging](https://github.com/vodafone/kramax-bpm-platform/tree/master/commons/logging) which itself uses [SLF4J](http://slf4j.org) as
a logging backend. To enable logging a SLF4J implementation has to be part of
your classpath. For example `slf4j-simple`, `log4j12` or `logback-classic`.

Apache HTTP Client 5.x has built-in SLF4J support, so no additional bridges are required for HTTP client logging.
