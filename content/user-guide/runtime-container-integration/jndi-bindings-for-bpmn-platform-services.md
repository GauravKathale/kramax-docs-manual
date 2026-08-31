---

title: 'JNDI Bindings for KramaX Services'
weight: 20

menu:
  main:
    identifier: "user-guide-runtime-container-integration-jndi"
    parent: "user-guide-runtime-container-integration"

---

The KramaX Services (i.e., Process Engine Service and Process Application Service) are provided via JNDI Bindings with the following JNDI names:

* Process Engine Service: `java:global/kramaX-bpm-platform/process-engine/ProcessEngineService!org.vodafone.kramaX.bpm.ProcessEngineService`
* Process Application Service: `java:global/kramaX-bpm-platform/process-engine/ProcessApplicationService!org.vodafone.kramaX.bpm.ProcessApplicationService`

On JBoss EAP and WildFly, you are able to get any of these KramaX Services through a JNDI lookup. 
However, on Apache Tomcat you have to do quite a bit more to be able to do a lookup to get one of these KramaX Services.
