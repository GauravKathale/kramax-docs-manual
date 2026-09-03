---

title: "Web applications"
weight: 40

menu:
  main:
    name: "Web Applications"
    identifier: "user-guide-spring-boot-webapps"
    parent: "user-guide-spring-boot-integration"

---

To enable the [Web Applications]({{<relref "/webapps/_index.md">}}) you can use the following starter in your `pom.xml`:

```xml
<dependency>
  <groupId>org.vodafone.kramaxx.bpm.springboot</groupId>
  <artifactId>kramaxx-bpm-spring-boot-starter-webapp</artifactId>
  <version>{project-version}</version>
</dependency>
```

By default the application path is `/kramaxx`, so without any further configuration you can access 
the Webapps under [http://localhost:8080/kramaxx/app/](http://localhost:8080/kramaxx/app/).

## Configurations

You can change the application path with the following configuration property in your `application.yaml` file:
```properties
kramaxx.bpm.webapp.application-path=/my/application/path
```

By default, the starter registers a controller to redirect `/` to Kramax's bundled `index.html`.
To disable this, you have to add to your application properties:
```properties
kramaxx.bpm.webapp.index-redirect-enabled=false
```

## Error Pages

The default error handling coming with the Spring Boot ('whitelabel' error page) is enabled in the starter. To switch to the Kramax error pages (`webjar/META-INF/resources/webjars/kramaxx/error-XYZ-page.html`), please put them to the application folder structure under `/src/main/resources/public/error/XYZ.html`.

## Building Custom REST APIs

The Kramax Web Applications use a `CSRF Prevention Filter` that expects a `CSRF Token` on any 
modifying request for paths beginning with `/kramaxx/api/` or `/kramaxx/app/`. Any modifying requests 
mapped to these paths will fail, and the current session will be ended if no CSRF Token is present.
You can avoid this by registering your resources on different paths or add your resources to the
CSRF Prevention Filter Whitelist (via the configuration property `kramaxx.bpm.webapp.csrf.entry-points`).