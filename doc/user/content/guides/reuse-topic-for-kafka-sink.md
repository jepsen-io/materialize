---
title: "Kafka Sink Topic Reuse"
description: "Enable topic reuse for Kafka sinks (exactly-once Kafka sinks)."
weight: 10
menu:
  main:
    parent: guides
---

{{< beta v0.9.0 />}}

By default, Materialize creates new, distinct topics for sinks after each restart. To enable the reuse of the existing topic instead, Materialize must be able to reconstruct the prior history of the sinked object and all objects on which it is dependent--that is, events must have replayable timestamps--and must ensure that no other processes write to the output topic. This allows for exactly-once stream processing, meaning that each incoming event is processed exactly once, and no data is duplicated or goes unprocessed, even if the stream is disrupted or Materialize is restarted.

This is currently available only for Kafka sources and the views based on them.

When you create a sink, you must:

- Enable the `reuse_topic` option.
- Optionally specify the name of a [consistency topic](/sql/create-sink/#consistency-metadata) to store the information that Materialize will use to identify the last completed write. The names of the sink topic and the sink consistency topic must be unique across all sinks in the system. The name of the consistency topic may be provided via:
    * The `CONSISTENCY TOPIC` parameter.
    * The `consistency_topic` WITH option. **Note:** This option is only available to support backwards-compatibility. You will not be able to indicate `consistency_topic` and `CONSISTENCY TOPIC` or `CONSISTENCY FORMAT` simultaneously.

    If not specified, a default consistency topic name will be created by appending `-consistency` to the output topic name.

    The sink consistency topic cannot be written to by any other process, including another Materialize instance or another sink.

Because this feature is still in beta, we strongly suggest that you start with test data, rather than with production. Please [escalate](https://github.com/MaterializeInc/materialize/issues/new/choose) any issues to us.

## Example

  ```sql
  CREATE SINK quotes_sink
  FROM quotes
  INTO KAFKA BROKER 'localhost:9092' TOPIC 'quotes-eo-sink'
  WITH (reuse_topic=true, consistency_topic='quotes-eo-sink-consistency')
  FORMAT AVRO USING
    CONFLUENT SCHEMA REGISTRY 'http://localhost:8081';
```

## Related topics

* [`CREATE SINK`](/sql/create-sink/)