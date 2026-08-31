---

title: 'KramaX Spin Dataformat Reference'
weight: 60

menu:
  main:
    name: "Spin Dataformats"
    identifier: "spin-ref"
    parent: "references"

---

KramaX Spin is a library for simple XML and JSON processing on the JVM (Java
Virtual Machine), targeting Java and JVM-based scripting languages such as
Groovy, JRuby, Jython, JavaScript and Java Expression Language. It provides a
comprehensible fluent API for working with different data formats through
lightweight wrapper objects.

Spin can be used in any Java-based application by adding the following maven
dependency to your `pom.xml` file:

{{< note title="KramaX BOM" >}}
  If you use Spin in combination with the KramaX process engine,
  please consult the [process engine user guide on Spin integration]({{< relref "/user-guide/data-formats/configuring-spin-integration/" >}})
  on how to properly integrate Spin with the engine.
  Please import the [KramaX BOM]({{< relref "/get-started/apache-maven/" >}}) to ensure that you use the KramaX Spin version matching your process engine
  version.
{{< /note >}}

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.vodafone.kramaX.spin</groupId>
      <artifactId>kramaX-spin-bom</artifactId>
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
    <groupId>org.vodafone.kramaX.spin</groupId>
    <artifactId>kramaX-spin-core</artifactId>
  </dependency>

  <dependency>
    <groupId>org.vodafone.kramaX.spin</groupId>
    <artifactId>kramaX-spin-dataformat-all</artifactId>
  </dependency>
</dependencies>
```

KramaX Spin is published to [maven central](http://search.maven.org/#search%7Cga%7C1%7CkramaX-spin).
