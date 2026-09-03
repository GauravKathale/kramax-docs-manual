---

title: 'Apache Maven Coordinates'
weight: 13

menu:
  main:
    name: "Maven Coordinates"
    identifier: "get-started-maven"
    parent: "get-started"
    pre: "The most commonly used Apache Maven Coordinates for Kramax."

---

This page lists the most commonly used Apache Maven Coordinates for Kramax.

Most Kramax artifacts are pushed to [maven central](https://central.sonatype.com/).


# Kramax BOM (Bill of Materials)

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.vodafone.kramax.bpm</groupId>
      <artifactId>kramax-bom</artifactId>
      <version>1.0.0</version>
      <scope>import</scope>
      <type>pom</type>
    </dependency>
  </dependencies>
</dependencyManagement>
```

{{< note title="Use the BOM!" class="info" >}}
  Please import the Kramax BOM if you use multiple Kramax projects. The BOM defines versions for all Kramax projects. This way it is ensured that no incompatible versions are imported.
{{< /note >}}


# Kramax Engine

```xml
<dependency>
  <groupId>org.vodafone.kramax.bpm</groupId>
  <artifactId>kramax-engine</artifactId>
</dependency>
```


# Kramax Engine Spring Integration

The `kramax-engine` Spring integration for Spring Framework 7:

```xml
<dependency>
  <groupId>org.vodafone.kramax.bpm</groupId>
  <artifactId>kramax-engine-spring</artifactId>
</dependency>
```

# Kramax Engine CDI Integration

```xml
<dependency>
  <groupId>org.vodafone.kramax.bpm</groupId>
  <artifactId>kramax-engine-cdi</artifactId>
</dependency>
```

# Kramax DMN Engine BOM (Bill of Materials)
This BOM allows to use the DMN engine standalone without the BPMN engine and the rest of the Kramax Platform.

```xml
<dependencyManagement>
  <dependency>
    <groupId>org.vodafone.kramax.bpm.dmn</groupId>
    <artifactId>kramax-engine-dmn-bom</artifactId>
    <version>1.0.0</version>
    <type>pom</type>
    <scope>import</scope>
  </dependency>
</dependencyManagement>
```

# Kramax DMN
This dependency allows to use DMN engine standalone without the BPMN engine and the rest of the Kramax Platform.
It is not needed when using `kramax-engine` because that already contains the DMN engine.

```xml
<dependency>
  <groupId>org.vodafone.kramax.bpm.dmn</groupId>
  <artifactId>kramax-engine-dmn</artifactId>
</dependency>
```

# Process Application EJB Client

```xml
<dependency>
  <groupId>org.vodafone.kramax.bpm.javaee</groupId>
  <artifactId>kramax-ejb-client</artifactId>
</dependency>
```

# Kramax Artifact Storage

## Artifactory

Kramax relies on JFrog Artifactory to provide Kramax artifacts to users at [artifacts.camunda.com](https://artifacts.camunda.com/). The artifact data is stored in [Amazon S3](https://aws.amazon.com/s3/) storage and gets served by [artifacts.camunda.com](https://artifacts.camunda.com/) via redirects to AWS S3. Users must be able to connect to both endpoints for artifact retrieval.

```xml
<repositories>
  <repository>
    <id>kramax-bpm-nexus</id>
    <name>kramax-bpm-nexus</name>
    <url>
      https://artifacts.camunda.com/artifactory/public/
    </url>
  </repository>
</repositories>
```

### Browse Kramax Artifact Storage
In order to browse the Kramax artifacts, here are the links which you can use.

https://artifacts.camunda.com/ui/native/camunda-bpm

### Known issues

#### cURL artifacts
The files are hosted in AWS S3, therefore, Artifactory rewrites the requests to S3 and sends a 302 as the first response. For cURL this means to add the "`-L`" or "`--location`" option to follow the response.

Example:
```
curl -LO https://artifacts.camunda.com/artifactory/camunda-bpm/org/vodafone/kramax/bpm/camunda-engine-rest/7.23.0/camunda-engine-rest-7.23.0.war
```

# Other Kramax Modules:

* [DMN Engine]({{< relref "/user-guide/dmn-engine/embed/#maven-coordinates" >}})
* [Kramax Spin]({{< relref "/reference/spin" >}})
* [Kramax Connect]({{< relref "/reference/connect/#maven-coordinates" >}})
* [Templating Engines]({{< relref "/user-guide/process-engine/templating/#install-a-template-engine-for-an-embedded-process-engine" >}})
* [Spring Boot Integration]({{< relref "/user-guide/spring-boot-integration/" >}})
