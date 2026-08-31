---

title: 'Apache Maven Coordinates'
weight: 13

menu:
  main:
    name: "Maven Coordinates"
    identifier: "get-started-maven"
    parent: "get-started"
    pre: "The most commonly used Apache Maven Coordinates for KramaX."

---

This page lists the most commonly used Apache Maven Coordinates for KramaX.

Most KramaX artifacts are pushed to [maven central](https://central.sonatype.com/).


# KramaX BOM (Bill of Materials)

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.vodafone.kramaX.bpm</groupId>
      <artifactId>kramaX-bom</artifactId>
      <version>1.0.0</version>
      <scope>import</scope>
      <type>pom</type>
    </dependency>
  </dependencies>
</dependencyManagement>
```

{{< note title="Use the BOM!" class="info" >}}
  Please import the KramaX BOM if you use multiple KramaX projects. The BOM defines versions for all KramaX projects. This way it is ensured that no incompatible versions are imported.
{{< /note >}}


# KramaX Engine

```xml
<dependency>
  <groupId>org.vodafone.kramaX.bpm</groupId>
  <artifactId>kramaX-engine</artifactId>
</dependency>
```


# KramaX Engine Spring Integration

The `kramaX-engine` Spring integration for Spring Framework 7:

```xml
<dependency>
  <groupId>org.vodafone.kramaX.bpm</groupId>
  <artifactId>kramaX-engine-spring</artifactId>
</dependency>
```

# KramaX Engine CDI Integration

```xml
<dependency>
  <groupId>org.vodafone.kramaX.bpm</groupId>
  <artifactId>kramaX-engine-cdi</artifactId>
</dependency>
```

# KramaX DMN Engine BOM (Bill of Materials)
This BOM allows to use the DMN engine standalone without the BPMN engine and the rest of the KramaX Platform.

```xml
<dependencyManagement>
  <dependency>
    <groupId>org.vodafone.kramaX.bpm.dmn</groupId>
    <artifactId>kramaX-engine-dmn-bom</artifactId>
    <version>1.0.0</version>
    <type>pom</type>
    <scope>import</scope>
  </dependency>
</dependencyManagement>
```

# KramaX DMN
This dependency allows to use DMN engine standalone without the BPMN engine and the rest of the KramaX Platform.
It is not needed when using `kramaX-engine` because that already contains the DMN engine.

```xml
<dependency>
  <groupId>org.vodafone.kramaX.bpm.dmn</groupId>
  <artifactId>kramaX-engine-dmn</artifactId>
</dependency>
```

# Process Application EJB Client

```xml
<dependency>
  <groupId>org.vodafone.kramaX.bpm.javaee</groupId>
  <artifactId>kramaX-ejb-client</artifactId>
</dependency>
```

# KramaX Artifact Storage

## Artifactory

KramaX relies on JFrog Artifactory to provide KramaX artifacts to users at [artifacts.camunda.com](https://artifacts.camunda.com/). The artifact data is stored in [Amazon S3](https://aws.amazon.com/s3/) storage and gets served by [artifacts.camunda.com](https://artifacts.camunda.com/) via redirects to AWS S3. Users must be able to connect to both endpoints for artifact retrieval.

```xml
<repositories>
  <repository>
    <id>kramaX-bpm-nexus</id>
    <name>kramaX-bpm-nexus</name>
    <url>
      https://artifacts.camunda.com/artifactory/public/
    </url>
  </repository>
</repositories>
```

### Browse KramaX Artifact Storage
In order to browse the KramaX artifacts, here are the links which you can use.

https://artifacts.camunda.com/ui/native/camunda-bpm

### Known issues

#### cURL artifacts
The files are hosted in AWS S3, therefore, Artifactory rewrites the requests to S3 and sends a 302 as the first response. For cURL this means to add the "`-L`" or "`--location`" option to follow the response.

Example:
```
curl -LO https://artifacts.camunda.com/artifactory/camunda-bpm/org/vodafone/kramaX/bpm/camunda-engine-rest/7.23.0/camunda-engine-rest-7.23.0.war
```

# Other KramaX Modules:

* [DMN Engine]({{< relref "/user-guide/dmn-engine/embed/#maven-coordinates" >}})
* [KramaX Spin]({{< relref "/reference/spin" >}})
* [KramaX Connect]({{< relref "/reference/connect/#maven-coordinates" >}})
* [Templating Engines]({{< relref "/user-guide/process-engine/templating/#install-a-template-engine-for-an-embedded-process-engine" >}})
* [Spring Boot Integration]({{< relref "/user-guide/spring-boot-integration/" >}})
