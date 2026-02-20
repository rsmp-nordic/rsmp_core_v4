---
title: History
parent: Messages
nav_order: 10
permalink: /messages/history/
---

# History
History messages are published by a node in response to a [Fetch](fetch.md)
request. They are delivered directly to the requesting supervisor via the MQTT 5
Response Topic set in the fetch message.

```
<supervisor>/history/<code>/<channel>[/<component>]
```

If the channel name is omitted in the corresponding status topic, it MUST also be omitted here:

```
<supervisor>/history/<code>
```

Examples:
```
22ba/history/tlc.groups/hourly              # history delivered to supervisor 22ba
22ba/history/tlc.plan                       # history for current plan (single channel, name omitted)
22ba/history/traffic.count/hourly/dl/1      # history for detector logic 1
```

The topic path is set by the supervisor as the Response Topic in the fetch
request. The supervisor MUST use this exact format, matching the `code`,
`channel`, and `component` from the fetch. This strict format is mandated for global observability and debugging by third-party clients. The node publishes to the Response
Topic verbatim without validation.

## MQTT Properties

| Property | Value |
|---|---|
| QoS | `1` |
| Retain | `false` |
| Correlation Data | MUST echo the value from the fetch request |

## Payload

History messages use the same attribute schema as the channel's normal status
messages, with additional metadata fields. The payload is CBOR encoded (represented here as JSON). Timestamps MUST be encoded as ISO 8601 strings.

```json
{
  "ts":       "2026-02-19T08:00:00Z",
  "values":   { ... },
  "seq":      101,
  "complete": false
}
```

| Field | Type | Description |
|---|---|---|
| `ts` | ISO 8601 timestamp | Original recording time on the device |
| `values` | object | Status attributes, identical in schema to live channel output |
| `seq` | integer | Original sequence number from the status channel |
| `complete` | boolean | `true` on the final message in the response |
| `truncated` | boolean | `true` if the node's buffer did not cover the full requested range (optional, included only when true) |

`seq` uses the same counter as live and replay messages for the same channel,
allowing the supervisor to locate history messages precisely within the overall
data series.

Consumers MUST use `ts` to position history messages in the time series, not
the MQTT message arrival time.

## Empty Response

If no data is available for the requested range, the node MUST send a single
message with `complete: true` and no `values`, `seq`, or `ts` fields:

```json
{
  "complete": true
}
```

## Supervisor Subscription

The supervisor MUST subscribe to its own history topic before issuing any
fetch requests:

```
<supervisor>/history/#
```

Correlation Data ties each history message back to the originating fetch,
allowing the supervisor to handle multiple concurrent fetches for the same or
different channels without confusion.
