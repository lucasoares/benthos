---
title: kafka
type: output
---

<!--
     THIS FILE IS AUTOGENERATED!

     To make changes please edit the contents of:
     lib/output/kafka.go
-->


The kafka output type writes a batch of messages to Kafka brokers and waits for
acknowledgement before propagating it back to the input.


import Tabs from '@theme/Tabs';

<Tabs defaultValue="common" values={[
  { label: 'Common', value: 'common', },
  { label: 'Advanced', value: 'advanced', },
]}>

import TabItem from '@theme/TabItem';

<TabItem value="common">

```yaml
output:
  kafka:
    addresses:
    - localhost:9092
    topic: benthos_stream
    client_id: benthos_kafka_output
    key: ""
    partitioner: fnv1a_hash
    compression: none
    max_in_flight: 1
    batching:
      count: 1
      byte_size: 0
      period: ""
```

</TabItem>
<TabItem value="advanced">

```yaml
output:
  kafka:
    addresses:
    - localhost:9092
    tls:
      enabled: false
      skip_cert_verify: false
      root_cas_file: ""
      client_certs: []
    sasl:
      mechanism: ""
      user: ""
      password: ""
      access_token: ""
      token_cache: ""
      token_key: ""
    topic: benthos_stream
    client_id: benthos_kafka_output
    key: ""
    partitioner: fnv1a_hash
    compression: none
    max_in_flight: 1
    ack_replicas: false
    max_msg_bytes: 1000000
    timeout: 5s
    target_version: 1.0.0
    batching:
      count: 1
      byte_size: 0
      period: ""
      condition:
        static: false
        type: static
      processors: []
    max_retries: 0
    backoff:
      initial_interval: 3s
      max_interval: 10s
      max_elapsed_time: 30s
```

</TabItem>
</Tabs>

The config field `ack_replicas` determines whether we wait for
acknowledgement from all replicas or just a single broker.

Both the `key` and `topic` fields can be dynamically set using
function interpolations described [here](/docs/configuration/interpolation#functions).
When sending batched messages these interpolations are performed per message
part.

## Performance

This output benefits from sending multiple messages in flight in parallel for
improved performance. You can tune the max number of in flight messages with the
field `max_in_flight`.

This output benefits from sending messages as a batch for improved performance.
Batches can be formed at both the input and output level. You can find out more
[in this doc](/docs/configuration/batching).

## Fields

### `addresses`

`array` A list of broker addresses to connect to. If an item of the list contains commas it will be expanded into multiple addresses.

```yaml
# Examples

addresses:
- localhost:9092

addresses:
- localhost:9041,localhost:9042

addresses:
- localhost:9041
- localhost:9042
```

### `tls`

`object` Custom TLS settings can be used to override system defaults.

### `tls.enabled`

`bool` Whether custom TLS settings are enabled.

### `tls.skip_cert_verify`

`bool` Whether to skip server side certificate verification.

### `tls.root_cas_file`

`string` The path of a root certificate authority file to use.

### `tls.client_certs`

`array` A list of client certificates to use.

```yaml
# Examples

client_certs:
- cert: foo
  key: bar

client_certs:
- cert_file: ./example.pem
  key_file: ./example.key
```

### `sasl`

`object` Enables SASL authentication.

### `sasl.mechanism`

`string` The SASL authentication mechanism, if left empty SASL authentication is not used. Warning: SCRAM based methods within Benthos have not received a security audit.

Options are: `PLAIN`, `OAUTHBEARER`, `SCRAM-SHA-256`, `SCRAM-SHA-512`.

### `sasl.user`

`string` A `PLAIN` username. It is recommended that you use environment variables to populate this field.

```yaml
# Examples

user: ${USER}
```

### `sasl.password`

`string` A `PLAIN` password. It is recommended that you use environment variables to populate this field.

```yaml
# Examples

password: ${PASSWORD}
```

### `sasl.access_token`

`string` A static `OAUTHBEARER` access token

### `sasl.token_cache`

`string` Instead of using a static `access_token` allows you to query a [`cache`](/docs/components/caches/about) resource to fetch `OAUTHBEARER` tokens from

### `sasl.token_key`

`string` Required when using a `token_cache`, the key to query the cache with for tokens.

### `topic`

`string` The topic to publish messages to.

This field supports [interpolation functions](/docs/configuration/interpolation#functions).

### `client_id`

`string` An identifier for the client connection.

### `key`

`string` The key to publish messages with.

This field supports [interpolation functions](/docs/configuration/interpolation#functions).

### `partitioner`

`string` The partitioning algorithm to use.

Options are: `fnv1a_hash`, `murmur2_hash`, `random`, `round_robin`.

### `compression`

`string` The compression algorithm to use.

Options are: `none`, `snappy`, `lz4`, `gzip`.

### `max_in_flight`

`number` The maximum number of parallel message batches to have in flight at any given time.

### `ack_replicas`

`bool` Ensure that messages have been copied across all replicas before acknowledging receipt.

### `max_msg_bytes`

`number` The maximum size in bytes of messages sent to the target topic.

### `timeout`

`string` The maximum period of time to wait for message sends before abandoning the request and retrying.

### `target_version`

`string` The version of the Kafka protocol to use.

### `batching`

`object` Allows you to configure a [batching policy](/docs/configuration/batching).

```yaml
# Examples

batching:
  byte_size: 5000
  period: 1s

batching:
  count: 10
  period: 1s

batching:
  condition:
    text:
      arg: END BATCH
      operator: contains
  period: 1m
```

### `batching.count`

`number` A number of messages at which the batch should be flushed. If `0` disables count based batching.

### `batching.byte_size`

`number` An amount of bytes at which the batch should be flushed. If `0` disables size based batching.

### `batching.period`

`string` A period in which an incomplete batch should be flushed regardless of its size.

```yaml
# Examples

period: 1s

period: 1m

period: 500ms
```

### `batching.condition`

`object` A [condition](/docs/components/conditions/about) to test against each message entering the batch, if this condition resolves to `true` then the batch is flushed.

### `batching.processors`

`array` A list of [processors](/docs/components/processors/about) to apply to a batch as it is flushed. This allows you to aggregate and archive the batch however you see fit. Please note that all resulting messages are flushed as a single batch, therefore splitting the batch into smaller batches using these processors is a no-op.

```yaml
# Examples

processors:
- archive:
    format: lines

processors:
- archive:
    format: json_array

processors:
- merge_json: {}
```

### `max_retries`

`number` The maximum number of retries before giving up on the request. If set to zero there is no discrete limit.

### `backoff`

`object` Control time intervals between retry attempts.

### `backoff.initial_interval`

`string` The initial period to wait between retry attempts.

### `backoff.max_interval`

`string` The maximum period to wait between retry attempts.

### `backoff.max_elapsed_time`

`string` The maximum period to wait before retry attempts are abandoned. If zero then no limit is used.

