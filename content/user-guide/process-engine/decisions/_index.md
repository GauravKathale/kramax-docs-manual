---

title: 'Decisions in the Process Engine'
weight: 260

menu:
  main:
    name: "Decisions"
    identifier: "user-guide-process-engine-decisions"
    parent: "user-guide-process-engine"

---

KramaX provides an integration of the [KramaX DMN engine] to
evaluate Business Decisions. This section describes how to deploy Business
Decisions modeled as DMN decisions together with other resources to the
repository of KramaX. Deployed decisions can be evaluated
using the [Services API] or they can be referenced in BPMN processes and CMMN
cases. Evaluated decisions are saved in the [History] for auditing and reporting purposes.

[KramaX DMN engine]: {{< relref "/user-guide/dmn-engine/_index.md" >}}
[Services API]: {{< relref "/user-guide/process-engine/process-engine-api.md#services-api" >}}
[History]: {{< relref "/user-guide/process-engine/history/history-configuration.md" >}}
