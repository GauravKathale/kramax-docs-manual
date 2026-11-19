---

title: 'Kramax Kafka Connector'
weight: 80

menu:
  main:
    name: "Kafka Connector"
    identifier: "user-guide-kafka-connector"
    parent: "plugins"

---

# Kramax Kafka Connector - User Guide

Outbound (producer) and inbound (consumer) connector - Kramax `1.0.0`

This guide describes the Kafka connector of the Kramax platform (`kramax-connect-kafka-client`) and the matching element templates of Kramax Modeler.

## 1. Overview

The Kafka connector lets processes exchange records with Apache Kafka without custom Java code.

- Outbound connector (`kafka-connector`, template `Kafka Outbound Connector`): a service/send task publishes one record and continues after broker acknowledgement (`acks=all`).
- Inbound connector (`KafkaInboundProcessEnginePlugin`, templates `Kafka Start Event Connector` and `Kafka Receive Connector`): consumers run in the engine and start instances or correlate messages per record.

| Use | When |
| --- | --- |
| Outbound | Process produces an event/command: notify systems, request work, publish state changes |
| Inbound - start event | Each record starts a new instance |
| Inbound - intermediate catch / receive task | A running instance waits for a reply/event identified by a correlation key |
| Inbound - boundary event | A running task is interrupted (or extended) by an event |
| Inbound - event sub process start | React to events during the full instance lifetime |

Delivery guarantees:

- Outbound is at-least-once. Keep async before/after enabled to avoid accidental duplicate side effects after transaction rollbacks.
- Inbound is at-least-once and ordered per partition. Offsets are committed after engine transaction commit.

### 1.1 Key features at a glance

- SASL/PLAIN, SCRAM-SHA-256, SCRAM-SHA-512, SSL, SASL_SSL, and mTLS with `{{secrets.NAME}}` placeholders.
- Keys, headers, explicit partition, tombstones, and extra Kafka client options via `additionalProperties`.
- JSON, inline Avro, and Confluent Schema Registry (Avro and JSON schema).
- Activation conditions, correlation by payload/business key, and result mapping using JUEL expressions.
- Retry/discard/dead-letter options for failed records plus self-resolving connection and dead-letter incidents.
- Several brokers, clusters, and topics per element, including multi-engine setups.

## 2. Prerequisites and enabling the connector

### 2.1 Runtime

| Distribution | What to do |
| --- | --- |
| Spring Boot application | Add `kramax-engine-plugin-connect` and `kramax-connect-kafka-client` (`1.0.0`). Plugins are auto-registered when classes are on the classpath. Startup log includes `CNCT-01004` for `kafka-connector`. |
| Tomcat / Run distributions | Shipped with distribution and auto-registered |
| WildFly | Inbound not covered |
| connectors-all | Kafka is intentionally not included; add `kramax-connect-kafka-client` explicitly |
| Multi-engine | Plugin runs on every engine |

```xml
<dependency>
  <groupId>org.vodafone.kramax.bpm</groupId>
  <artifactId>kramax-engine-plugin-connect</artifactId>
  <version>1.0.0</version>
</dependency>
<dependency>
  <groupId>org.vodafone.kramax.connect</groupId>
  <artifactId>kramax-connect-kafka-client</artifactId>
  <version>1.0.0</version>
</dependency>
```

### 2.2 Modeler

Templates are in `kramax-modeler/resources/platform/base/resources/element-templates`:

- `kafka-outbound-connector.json`
- `kafka-inbound-connector.json`

Select an element and choose Template -> `Kafka Outbound Connector`, `Kafka Start Event Connector`, or `Kafka Receive Connector`.

The older `Kafka Message Start Event Connector` is deprecated. Existing elements still run (warning `KFKA-03029`), but re-apply `Kafka Start Event Connector`.

Modeler saves extension elements with the `kramax:` namespace (`http://kramax.vodafone.org/schema/1.0/bpmn`); the engine also accepts the `camunda:` namespace.

### 2.3 Secrets

Any connection/credential/registry/additionalProperties value may contain `{{secrets.NAME}}`.

Runtime replacement:

- env var `KRAMAX_SECRET_NAME` (uppercase, `.` and `-` become `_`)
- system property `kramax.secret.NAME`

Only `KRAMAX_SECRET_` variables are accessible to process models.

```bash
export KRAMAX_SECRET_KAFKA_USER=<user>
export KRAMAX_SECRET_KAFKA_PASSWORD=<password>
java -jar your-application.jar
# or: -Dkramax.secret.KAFKA_PASSWORD=<password>
```

### 2.4 Engine settings

- Job executor enabled (default in Spring Boot).
- Inbound consumers follow latest active process version; reconcile interval is 5s.
- Enable debug logging when needed:

```properties
logging.level.org.vodafone.kramax.bpm.connect=DEBUG
```

### 2.5 Kafka concepts used in this guide

| Term | Meaning for the connector |
| --- | --- |
| Broker / bootstrap servers | Kafka servers (`host:port`, comma-separated); one reachable broker is enough to discover the cluster |
| Topic | Named stream where records are produced/consumed |
| Partition | Ordered slice of a topic; order is guaranteed only within a partition |
| Record | Key, value, headers, topic, partition, offset, and timestamp |
| Offset | Record position in a partition |
| Consumer group | Consumer set sharing partitions for a topic |
| Tombstone | Record with null value, usually for delete semantics in compacted topics |

## 3. Kafka connection and authentication

Connection settings are shared between outbound and inbound elements.

| Option | Settings |
| --- | --- |
| No auth (PLAINTEXT) | `authenticationType=none`, `securityProtocol` empty |
| SASL/PLAIN | `authenticationType=plain`, `username`, `password`, `securityProtocol=SASL_PLAINTEXT` or `SASL_SSL` |
| SASL/SCRAM-SHA-256 | `authenticationType=scram-sha-256` + `username/password` |
| SASL/SCRAM-SHA-512 | `authenticationType=scram-sha-512` + `username/password` |
| TLS (SSL) | `securityProtocol=SSL`; truststore via `additionalProperties` |
| SASL over TLS | auth type set, `securityProtocol` empty (defaults to `SASL_SSL`) + truststore |
| mTLS | `securityProtocol=SSL` + `ssl.keystore.*` in `additionalProperties` |
| OAUTHBEARER / AWS IAM / Kerberos | `authenticationType=none` + `sasl.*` in `additionalProperties` |

### 3.1 Several brokers and clusters

- Bootstrap list can include several brokers: `broker-1:9092,broker-2:9092,broker-3:9092`.
- Each connector element has independent connection settings, so one process can publish/consume across different clusters.
- With replication factor 3, `min.insync.replicas=2`, and default `acks=all`, one broker can fail without data loss.

TLS truststore example (`additionalProperties`):

```properties
ssl.truststore.type=PKCS12
ssl.truststore.location={{secrets.KAFKA_TRUSTSTORE_PATH}}
ssl.truststore.password={{secrets.KAFKA_TRUSTSTORE_PASSWORD}}
```

The connector builds `sasl.jaas.config` from username/password (with quote escaping). Kafka logs it as `[hidden]`.

## 4. Outbound connector

Apply `Kafka Outbound Connector` to a service task or send task.

### 4.1 Input parameters

| Parameter | Type | Default | Required | Description |
| --- | --- | --- | --- | --- |
| `bootstrapServers` | String | - | Yes* | Comma-separated broker list, for example `broker-1:9092,broker-2:9092`; supports `{{secrets.*}}`. Optional only when `additionalProperties` sets `bootstrap.servers`. |
| `securityProtocol` | Dropdown | empty | No | `PLAINTEXT`, `SSL`, `SASL_PLAINTEXT`, `SASL_SSL`. Empty means `SASL_SSL` when authentication is set, else Kafka default (`PLAINTEXT`). |
| `authenticationType` | Dropdown | `none` | No | `none`, `plain`, `scram-sha-256`, `scram-sha-512`. If missing and both username/password are present, `plain` is used. |
| `username` | String | - | With SASL | SASL user; supports `{{secrets.*}}`. |
| `password` | String | `{{secrets.KAFKA_PASSWORD}}` | With SASL | SASL password; prefer secrets placeholder. |
| `topic` | String | - | Yes | Target topic; expressions allowed. |
| `key` | String / expression | - | No | Record key, for example `${orderId}`. Maps/lists are serialized as JSON; byte arrays are sent as-is. |
| `value` | Text / expression / map | - | No | Text/JSON text is UTF-8; maps/lists are serialized as JSON; byte arrays are sent as-is. Empty value sends tombstone (null). |
| `headers` | Text / map | - | No | JSON object (for example `{"source":"kramax"}`) or map. Values are sent as UTF-8 strings; expressions inside JSON are resolved. |
| `partition` | Integer | - | No | Explicit partition. Empty lets the partitioner decide (usually key hash). |
| `schemaStrategy` | Dropdown | `noSchema` | No | `noSchema`, `inlineSchema` (Avro binary), `schemaRegistry` (Confluent wire format). |
| `schema` | Text | - | inlineSchema | Avro schema JSON. With `schemaRegistry`, optional; empty uses latest subject version. |
| `schemaType` | Dropdown | `AVRO` | No | Registry mode only: `AVRO` or `JSON`. |
| `schemaRegistryUrl` | String | - | schemaRegistry | Example: `http://schema-registry:8081`; supports `{{secrets.*}}`. |
| `schemaSubject` | String | `<topic>-value` | No | Registry subject name. |
| `autoRegisterSchema` | Dropdown | `false` | No | `true` registers provided schema under subject. |
| `schemaRegistryUsername` / `schemaRegistryPassword` | String | - | No | Basic auth for schema registry; supports `{{secrets.*}}`. |
| `additionalProperties` | Text / map | - | No | Kafka producer properties as JSON or `key=value` lines. Overrides other settings. Blocked: `key.serializer`, `value.serializer`, `transactional.id`. |
| `errorCode` | String | - | No | If set, failures raise BPMN error code (catch via error boundary event). |
| `errorScope` | Dropdown | `permanent` | No | `permanent`: only non-retryable failures raise BPMN error; retryable failures use job retries. `all`: every failure raises BPMN error and skips job retry loop. |
| Asynchronous before / after | Boolean | `true` / `true` | No | Keep enabled so connector failures use job retries/incidents and post-send failures do not cause hidden duplicate side effects. |

### 4.2 Output

| Response parameter | Type | Description |
| --- | --- | --- |
| `topic` | String | Written topic |
| `partition` | Integer | Partition |
| `offset` | Long | Offset |
| `timestamp` | Long | Broker/create timestamp (epoch ms) |
| `response` | String (JSON) | Combined response payload |

Template maps `response` to `kafkaResponse`. Example access:

```java
${S(kafkaResponse).prop("offset").numberValue()}
```

### 4.3 Producer defaults

| Property | Value |
| --- | --- |
| `acks` | `all` |
| `enable.idempotence` | `true` (auto-disabled if `acks` overridden to `0/1` unless explicitly set) |
| `delivery.timeout.ms` | `45000` |
| `max.block.ms` | `15000` |
| `client.dns.lookup` | `use_all_dns_ips` |
| Producer pool | one producer per distinct configuration, max 16, closed after 10 min idle |

### 4.4 Error handling

| Situation | Behavior |
| --- | --- |
| Retryable failure (timeout, broker down, missing topic) | Job fails and retries according to `failedJobRetryTimeCycle` (default usually 3 total attempts), then failed-job incident |
| Permanent failure (serialization, authentication, invalid config), no `errorCode` | Same job retry/incident behavior |
| Permanent failure with `errorCode` | BPMN error is raised; message appears in error message variable |
| `errorScope=all` | Every failure raises BPMN error immediately; no job retries |

### 4.5 Outbound BPMN example

```xml
<bpmn:serviceTask id="Publish" name="Publish order event" kramax:asyncBefore="true" kramax:asyncAfter="true"
    kramax:modelerTemplate="org.vodafone.kramax.connector.kafka.outbound">
  <bpmn:extensionElements><kramax:connector><kramax:inputOutput>
    <kramax:inputParameter name="bootstrapServers">broker-1:9092,broker-2:9092</kramax:inputParameter>
    <kramax:inputParameter name="topic">orders</kramax:inputParameter>
    <kramax:inputParameter name="key">${orderId}</kramax:inputParameter>
    <kramax:inputParameter name="value">{"orderId":"${orderId}","status":"CREATED"}</kramax:inputParameter>
    <kramax:outputParameter name="kafkaResponse">${response}</kramax:outputParameter>
  </kramax:inputOutput><kramax:connectorId>kafka-connector</kramax:connectorId></kramax:connector>
  </bpmn:extensionElements>
</bpmn:serviceTask>
```

## 5. Inbound connector

| Template | Applies to | Behavior |
| --- | --- | --- |
| `Kafka Start Event Connector` | Start event (none/message), event sub process start | Starts instances from records |
| `Kafka Receive Connector` | Receive task, message catch/boundary events | Correlates records to waiting instances |

Elements need a BPMN message except for none start events.

### 5.1 Create an inbound element

1. Add a start event, message intermediate catch event, receive task, or message boundary event.
2. Apply `Kafka Start Event Connector` (start events) or `Kafka Receive Connector` (other inbound waits).
3. Select or create BPMN Message (none start events can skip this).
4. Fill Kafka cluster/authentication/topic values.
5. Set consumer group and offset behavior if defaults do not fit.
6. For catch/receive/boundary, configure correlation key expression and matching process variable.
7. Optionally configure activation condition, result mapping, and failure handling, then deploy. Consumer starts within reconcile interval.

### 5.2 Properties

| Property | Type | Default | Required | Description |
| --- | --- | --- | --- | --- |
| `inbound.type` | Hidden | `kafka` | Yes | Runtime marker. Elements without it are ignored, except legacy template compatibility. |
| `bootstrapServers` | String | `localhost:9092` | Yes | Broker list; same semantics as outbound. |
| `topic` | String | - | Yes | Single topic or comma-separated topic list. |
| `groupId` | String | `kramax-inbound-<process>-<element>` | No | Consumer group id; stable default across process versions. |
| `autoOffsetReset` | Dropdown | `latest` | No | Start behavior when no committed offset exists: `latest`, `earliest`, `none`. |
| `offsets` | String | - | No | Comma-separated per-partition start offsets (index = partition number). Used only when no committed offsets exist for that partition. |
| `authenticationType`, `securityProtocol`, `username`, `password` | - | `none` | No | Same authentication model as outbound. |
| `schemaStrategy`, `schema`, `schemaRegistryUrl`, `schemaRegistryUsername/Password` | - | `noSchema` | No | Same schema model as outbound. For `schemaRegistry`, schema is looked up by id in the record payload. |
| `additionalProperties` | Text | - | No | Kafka consumer properties. Blocked: `key.deserializer`, `value.deserializer`, `enable.auto.commit`. |
| `activationCondition` | JUEL expression | - | No | Boolean expression, for example `${value.type == 'ORDER'}`. False means unmatched record. |
| `consumeUnmatchedEvents` | Dropdown | Start: `true`, Receive: `false` | No | `true`: unmatched records are skipped + committed. `false`: unmatched records flow to failure handling. |
| `businessKeyExpression` | Expression | - | No | Start events only; business key for created instance. |
| `correlationKeyExpression` | Expression | - | No | Catch/receive/boundary: key derived from record. `null` means unmatched. Empty can correlate all waiting instances for message. |
| `correlationVariable` | String | - | No | Variable to match against correlation key. Empty means compare against business key. |
| `resultVariable` | String | `kafkaMessage` | No | Stores key/value/headers/topic/partition/offset/timestamp as JSON text. |
| `resultExpression` | Text | - | No | JSON object or `name=expression` lines; maps/lists are stored as JSON text. |
| `failureHandling` | Dropdown | Start: `retry`, Receive: `discard` | No | `retry`, `discard`, `deadLetter`. |
| `retries` | Integer | `3` (Receive template: `10`) | No | Attempts after first processing attempt (for `discard`/`deadLetter`). |
| `retryBackoffMillis` | Integer | `1000` | No | Initial retry delay; doubles up to 60s cap. |
| `deadLetterTopic` | String | - | with `deadLetter` | Must exist; otherwise `kafkaDeadLetter` incident is raised. |
| `connectionRetries` | Integer | `3` | No | Connection attempts after first before raising `kafkaConnection` incident. |
| `connectionRetryIntervalMillis` | Integer | `10000` | No | Wait interval between connection checks. |

### 5.3 Expression context

| Variable | Content |
| --- | --- |
| `key` | Key as UTF-8 text |
| `value` | Decoded payload (JSON maps/lists, Avro map, or text) |
| `rawValue` | Raw UTF-8 text |
| `headers` | Header map |
| `topic`, `partition`, `offset`, `timestamp` | Record coordinates |

Expressions are JUEL (`${...}`).

### 5.4 How a record is processed

1. Activation condition is evaluated. `false` means unmatched.
2. Start event: instance is created (business key from `businessKeyExpression` when configured).
3. Catch/receive/boundary: correlation key is computed and matched to waiting instances.
4. Unmatched handling:
   - `consumeUnmatchedEvents=true`: skip and commit.
   - `consumeUnmatchedEvents=false`: apply `failureHandling` (`retry`/`discard`/`deadLetter`).
5. For `retry`, partition stays blocked until success or strategy switch.

### 5.5 Consumer groups, offsets, versions

- Default `groupId`: `kramax-inbound-<processKey>-<elementId>`.
- Offsets commit after engine transaction (`enable.auto.commit=false`).
- New process versions continue with same group offsets.
- Latest process version consumes; older waiting instances can still be correlated when topic/message align.
- In cluster mode, nodes share group id and Kafka rebalances partitions across nodes.

### 5.6 Result mapping

```properties
resultVariable = kafkaMessage
resultExpression = orderId=${value.orderId}
source=${headers.source}
recordOffset=${offset}
```

### 5.7 Dead letters

Dead-lettered records keep original headers and add:

- `kramax.dlt.topic`
- `kramax.dlt.partition`
- `kramax.dlt.offset`
- `kramax.dlt.reason`

If dead-letter topic is unavailable, partition remains held and `kafkaDeadLetter` incident appears until next successful dead-letter/process attempt.

## 6. Serialization and Schema Registry

| Strategy | Outbound | Inbound |
| --- | --- | --- |
| `noSchema` | UTF-8 text; maps/lists to JSON | JSON -> map/list, otherwise text |
| `inlineSchema` | Avro binary with supplied schema | Avro decoded with supplied schema |
| `schemaRegistry` AVRO | Confluent wire format (magic byte + schema id + Avro) | Lookup schema by id |
| `schemaRegistry` JSON | Magic byte + schema id + JSON | Decode as JSON |

## 7. Headers, keys, and additional properties

- Headers can be JSON object or map; values sent as UTF-8 strings.
- Key controls partition locality with default partitioner; explicit `partition` overrides.
- `additionalProperties` supports JSON or key/value lines and overrides standard fields.
- Blocked managed properties:
  - producer: `key.serializer`, `value.serializer`, `transactional.id`
  - consumer: `key.deserializer`, `value.deserializer`, `enable.auto.commit`

Example:

```properties
compression.type=lz4
linger.ms=5
client.id=order-service
acks=1
```

## 8. Error handling, retries, incidents, troubleshooting

### 8.1 Typical behavior matrix

| Situation | Behavior |
| --- | --- |
| Outbound retryable failure | Job retries (`failedJobRetryTimeCycle`) then incident |
| Outbound permanent failure + `errorCode` | BPMN error raised |
| Inbound connection lost | `kafkaConnection` incident, auto-resolves on reconnect |
| Inbound record failure | `retry` / `discard` / `deadLetter` handling |
| Dead-letter topic unavailable | `kafkaDeadLetter` incident; partition held until resolved |

### 8.2 Common troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| No inbound subscription started | Missing `inbound.type`, inactive latest version, invalid config | Re-apply template and review `KFKA-03015` logs |
| Topic metadata timeout | Missing topic or broker unreachable | Check topic existence and network/listeners |
| Secret missing incident | Missing `KRAMAX_SECRET_*` variable | Export secret and restart |
| Auth failure | Wrong credentials/mechanism/listener | Align connector auth with broker config |
| Correlation fails repeatedly | No waiting instance or type mismatch | Validate key mapping and variable types |

### 8.3 Log codes (selected)

| Code | Meaning |
| --- | --- |
| `KFKA-03001` | Published record (DEBUG) |
| `KFKA-03005/006` | Missing / invalid parameter |
| `KFKA-03007` | Secret not configured |
| `KFKA-03008` | Managed property in `additionalProperties` |
| `KFKA-03009` | Publish failed |
| `KFKA-03010/012` | Serialization / deserialization failed |
| `KFKA-03011` | Schema registry request failed |
| `KFKA-03013/014` | Inbound subscription started/stopped |
| `KFKA-03015` | Inbound subscription not started (invalid settings) |
| `KFKA-03017/018` | Record delivered / skipped (DEBUG) |
| `KFKA-03019` | Record failed, retrying |
| `KFKA-03020/021` | Record discarded / dead-lettered |
| `KFKA-03025/026/027` | Connection failed / incident raised / connected again |
| `KFKA-03029` | Deprecated legacy start template in use (WARN) |
| `KFKA-03030/031` | Dead-letter incident raised/resolved |

## 9. Security best practices

- Keep credentials out of BPMN and use `{{secrets.*}}` placeholders.
- Provide `KRAMAX_SECRET_*` from secure platform secrets (for example Kubernetes/Vault).
- Use `SASL_SSL` or `SSL` in production.
- Grant minimum required ACLs.
- Protect deployment rights and engine REST API; deployers can access secret placeholders.
- Apply strict ACL/retention to dead-letter topics.

## 10. Comparison with Camunda 8 and limitations

### 10.1 Comparison with Camunda 8 Kafka connector

| Camunda 8 feature | Kramax | Implementation |
| --- | --- | --- |
| Bootstrap servers, topic, key, value | Supported | `bootstrapServers`, `topic`, `key`, `value`; maps/lists sent as JSON |
| Headers | Supported | `headers` as JSON/map, UTF-8 strings |
| Partition selection | Supported (extra) | `partition` parameter |
| Tombstone | Supported (extra) | Empty `value` |
| Authentication: credentials (SASL PLAIN) | Supported | `authenticationType=plain`, `username`, `password` |
| SCRAM-SHA-256 / 512 | Supported (extra) | `authenticationType=scram-sha-256/scram-sha-512` |
| Security protocol | Supported | `securityProtocol`; defaults to `SASL_SSL` when authenticating |
| SSL / mTLS | Partial | Via `additionalProperties` (`ssl.truststore.*`, `ssl.keystore.*`) |
| Connection credential (8.10) | Not supported | Use `{{secrets.*}}` |
| Secrets | Supported | `{{secrets.NAME}}` -> `KRAMAX_SECRET_NAME` or `-Dkramax.secret.NAME` |
| Additional properties | Supported | JSON or `key=value`; managed serializer/transaction/autocommit properties blocked |
| Inline Avro schema | Supported | `schemaStrategy=inlineSchema` + `schema` |
| Schema Registry (Confluent) | Supported | `schemaStrategy=schemaRegistry`; `AVRO`/`JSON`; subject; optional auto-register |
| Result variable / FEEL result expression | Partial | Outbound output params (`topic/partition/offset/timestamp/response`), inbound `resultVariable` + `resultExpression` (JUEL) |
| Error expression / retries | Partial | `errorCode` + `errorScope` or engine job retries/incidents |
| FEEL expressions | Not supported | JUEL `${...}` only |
| Inbound message start event | Supported | `Kafka Start Event Connector` |
| Inbound intermediate catch / receive task | Supported | `Kafka Receive Connector` |
| Inbound boundary event | Supported | `Kafka Receive Connector` on message boundary |
| Inbound event sub process start | Supported | Start template in event sub process |
| Activation condition | Supported | `activationCondition` (JUEL) |
| Consume unmatched events | Supported | `consumeUnmatchedEvents=true/false` |
| Correlation key (payload/process) | Supported | `correlationKeyExpression` + `correlationVariable` or business key |
| Message ID deduplication | Not supported | Design idempotent flow |
| Message TTL | Not supported | Closest is unmatched retry flow |
| Consumer group id | Supported | `groupId`, stable default across versions |
| Offsets and auto offset reset | Supported | `offsets` + `autoOffsetReset` |
| Mapping context vars | Supported | `key`, `value`, `rawValue`, `headers`, `topic`, `partition`, `offset`, `timestamp` |
| Failure handling / dead letter | Supported (extra) | `retry/discard/deadLetter`, retries, backoff, dead-letter topic |
| Connection incident | Supported (extra) | `kafkaConnection`, auto-resolving |
| Dead-letter incident | Supported (extra) | `kafkaDeadLetter`, auto-resolving |
| Multiple topics | Supported (extra) | Comma-separated topic list |
| Multiple bootstrap servers | Supported | Comma-separated list; unreachable entries skipped |
| Several clusters in one process | Supported | Per-element connection settings |
| Broker failover | Supported | Standard Kafka client failover |
| Multi-engine | Supported | Plugin active on every engine |
| Connection test button | Not supported | - |

### 10.2 Known limitations

- Expressions are JUEL, not FEEL.
- No message-id deduplication: design idempotent processes.
- No message TTL buffering semantics.
- Advanced TLS/mTLS and non-plain SCRAM/SASL flows rely on `additionalProperties`.

## 11. End-to-end example: order round trip

Two processes communicate through two topics: the order process publishes a fulfilment request and waits; the fulfilment process starts from that request and publishes the result.

1. `Order placed` starts the order process with `orderId` and `customer`.
2. `Publish fulfilment request` (Kafka Outbound) writes to `qa.e2e.requests` with key `${orderId}` and payload `{"orderId":"${orderId}","customer":"${customer}"}`.
3. `Wait for fulfilment` (Kafka Receive) waits on `qa.e2e.fulfilled` and correlates `${value.orderId}` to `orderId`.
4. `Fulfilment requested` (Kafka Start Event) consumes `qa.e2e.requests` and starts one fulfilment instance per record.
5. `Publish fulfilment result` writes `{"orderId":"${orderId}","status":"FULFILLED"}` to `qa.e2e.fulfilled`.
6. Waiting order instance receives and maps fulfilment values, then completes.

Start event (fulfilment process):

```xml
<kramax:property name="inbound.type" value="kafka" />
<kramax:property name="bootstrapServers" value="localhost:29092" />
<kramax:property name="topic" value="qa.e2e.requests" />
<kramax:property name="authenticationType" value="none" />
<kramax:property name="schemaStrategy" value="noSchema" />
<kramax:property name="autoOffsetReset" value="earliest" />
<kramax:property name="businessKeyExpression" value="${key}" />
<kramax:property name="resultVariable" value="kafkaMessage" />
<kramax:property name="resultExpression" value="orderId=${value.orderId}&#10;correlationId=${headers.correlationId}" />
```

Intermediate catch event (order process):

```xml
<kramax:property name="inbound.type" value="kafka" />
<kramax:property name="bootstrapServers" value="localhost:29092" />
<kramax:property name="topic" value="qa.e2e.fulfilled" />
<kramax:property name="autoOffsetReset" value="earliest" />
<kramax:property name="correlationKeyExpression" value="${value.orderId}" />
<kramax:property name="correlationVariable" value="orderId" />
<kramax:property name="consumeUnmatchedEvents" value="false" />
<kramax:property name="failureHandling" value="retry" />
<kramax:property name="resultExpression" value="fulfilmentStatus=${value.status}&#10;fulfilledBy=${headers.source}" />
```

## 12. More tested scenarios

| Area | Scenarios |
| --- | --- |
| Outbound | OUT-01 basic publish, 02 key, 03 headers, 04 expression value, 05 Avro+registry, 06 producer properties, 07a-e auth (PLAIN/SCRAM/SSL/SASL_SSL), 08 secrets, 09 result mapping, 10 explicit partition, 11 tombstone, 12 BPMN error, 13 inline Avro, 14 registry JSON schema, 15 Avro int overflow |
| Inbound | IN-01 message start, 02 intermediate correlation, 03 boundary event, 04 activation condition, 05 message-id dedup check, 06 offset reset, 07 key/headers/value, 08 Avro deserialization, 09 dead letter, 10 discard, 11 initial offsets, 12 multiple topics, 13 receive task, 14 event sub process, 15 business key, 16 version offsets, 17 early record retry, 18 non-default engine |
| Multi-cluster | MULTI-01 bootstrap list, 02 cross-cluster multi-topic, 03 same topic on two clusters, 04 broker failover |
| Negative | NEG-01 broker unreachable, 02 missing topic, 03 wrong credentials, 04 malformed payload, 05 retry incident recovery, 06 restart without loss, 07 legacy template, 08 unknown secret, 09 managed property, 10 dead-letter topic missing |

## 13. Troubleshooting and FAQ

Most issues appear as outbound job incidents, inbound `kafkaConnection`/`kafkaDeadLetter` incidents, or `KFKA-03xxx` log entries.

### 13.1 Troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| No inbound subscription start log | Missing `inbound.type=kafka`, not latest active definition, or invalid settings (`KFKA-03015`) | Re-apply Start/Receive template, verify logs, activate definition |
| `Topic X not present in metadata` | Missing topic or broker unreachable | Check topic existence and bootstrap/network/listener settings |
| `Must set acks to all ... idempotent producer` | `acks` overridden with explicit `enable.idempotence=true` | Use `acks=all` or remove explicit idempotence override |
| `Secret 'NAME' is not configured` | Missing `KRAMAX_SECRET_NAME` | Export secret env var or pass `-Dkramax.secret.NAME` |
| Managed serializer/transactional property incident | Blocked property in `additionalProperties` | Remove blocked property |
| Authentication failure | Wrong credentials, mechanism, or listener type | Align connector auth with broker mechanism and listener |
| `kafkaConnection` incident | Consumer cannot connect after retries | Fix connectivity; incident auto-resolves after reconnect |
| Repeated correlation failures (`KFKA-03019`) | No waiting instance or key type mismatch | Validate correlation value/type and keep retry for early records |
| `kafkaDeadLetter` incident and stuck partition | Missing DLQ topic or missing write ACL | Create topic/grant ACL; next retry resolves incident |
| Duplicate instances | At-least-once delivery or duplicate upstream send | Make flow idempotent and enforce unique business keys |
| `Unable to deserialize message value` | Wrong schema strategy or schema mismatch | Align producer/consumer strategy and consider `deadLetter` |
| Invalid JSON treated as text | `noSchema` fallback behavior | Add activation validation (for example `${value.orderId != null}`) |
| Unexpected empty value | Tombstone sent (empty value) | Set explicit value when tombstone is not intended |

### 13.2 Best practices

- Keep async before/after enabled on outbound connector tasks.
- Use stable keys for ordering-sensitive workloads (for example `${orderId}`).
- Use `deadLetter` for poison-message scenarios and `retry` where loss is unacceptable.
- Set explicit `groupId` when offset-sharing/reset behavior must be controlled.
- Enable connector debug logs during setup, then reduce log level for normal operation.

### 13.3 FAQ

| Question | Answer |
| --- | --- |
| Does a new process version re-read topics? | No. Default `groupId` remains stable and continues from committed offsets. |
| How can I replay from earliest data? | Use a new `groupId` with `autoOffsetReset=earliest`, or configure initial offsets for a group with no commits. |
| Can one process use two Kafka clusters? | Yes. Each connector element has its own connection settings. |
| Can I use FEEL expressions from Camunda 8? | No. Rewrite as JUEL, for example `=value.orderId` -> `${value.orderId}`. |
| What happens while the engine is down? | Records remain in Kafka; consumers resume from committed offsets on restart. |
| Can I access full inbound record data? | Yes. `resultVariable` (default `kafkaMessage`) includes key, value, headers, topic, partition, offset, and timestamp. |


