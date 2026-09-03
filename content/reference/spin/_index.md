---

title: 'Kramax Spin Dataformat Reference'
weight: 60

menu:
  main:
    name: "Spin Dataformats"
    identifier: "spin-ref"
    parent: "references"

---

Kramax Spin is a library for simple XML and JSON processing on the JVM (Java
Virtual Machine), targeting Java and JVM-based scripting languages such as
Groovy, JRuby, Jython, JavaScript and Java Expression Language. It provides a
comprehensible fluent API for working with different data formats through
lightweight wrapper objects.

Spin can be used in any Java-based application by adding the following maven
dependency to your `pom.xml` file:

{{< note title="Kramax BOM" >}}
  If you use Spin in combination with the Kramax process engine,
  please consult the [process engine user guide on Spin integration]({{< relref "/user-guide/data-formats/configuring-spin-integration/" >}})
  on how to properly integrate Spin with the engine.
  Please import the [Kramax BOM]({{< relref "/get-started/apache-maven/" >}}) to ensure that you use the Kramax Spin version matching your process engine
  version.
{{< /note >}}

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.vodafone.kramax.spin</groupId>
      <artifactId>kramax-spin-bom</artifactId>
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
    <groupId>org.vodafone.kramax.spin</groupId>
    <artifactId>kramax-spin-core</artifactId>
  </dependency>

  <dependency>
    <groupId>org.vodafone.kramax.spin</groupId>
    <artifactId>kramax-spin-dataformat-all</artifactId>
  </dependency>
</dependencies>
```

Kramax Spin is published to [maven central](http://search.maven.org/#search%7Cga%7C1%7Ckramax-spin).
