---

title: 'Public API'
weight: 80

menu:
  main:
    identifier: "user-guide-introduction-public-api"
    parent: "user-guide-introduction"

---


KramaX provides a public API. This section covers the definition of the public API and backwards compatibility for version updates.


# Definition of Public API

The KramaX public API is limited to the following items:

Java API: 

All non-implementation Java packages (package name does not contain `impl`) of the following modules.

* `kramaX-engine`
* `kramaX-engine-spring`
* `kramaX-engine-cdi`
* `kramaX-engine-dmn`
* `kramaX-bpmn-model`
* `kramaX-cmmn-model`
* `kramaX-dmn-model`
* `kramaX-spin-core`
* `kramaX-connect-core`
* `kramaX-commons-typed-values`

HTTP API (REST API):

* `kramaX-engine-rest`: HTTP interface (set of HTTP requests accepted by the REST API as documented in [REST API reference]({{< relref "/reference/rest/_index.md" >}}). Java classes are not part of the public API.


# Backwards Compatibility for Public API

The KramaX versioning scheme follows the MAJOR.MINOR.PATCH pattern put forward by [Semantic Versioning](http://semver.org/). KramaX will maintain public API backwards compatibility for MINOR version updates. Example: Update from version `1.1.x` to `1.2.x` will not break the public API.
