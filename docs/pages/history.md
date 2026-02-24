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
`channel`, and `component` from the fetch. The node publishes to the Response
Topic verbatim without validation.

## MQTT Properties

| Property | Value |
|---|---|
| QoS | `1` |
| Retain | `false` |
| Correlation Data | MUST echo the value from the fetch request |

## Payload

History messages use the same attribute schema as the channel's normal status
messages, with additional metadata fields.
Timestamps MUST be encoded as ISO 8601 strings.

### Unbatched History Payload

```json
{
  "ts":       "2026-02-19T08:00:00Z",
  "next_ts":  "2026-02-19T08:00:05Z",
  "values":   { ... },
  "seq":      101,
  "complete": false
}
```

### Batched History Payload

A node MAY send multiple events in a single MQTT message using an `entries`
array, reducing network overhead during large history responses:

```json
{
  "entries": [
    {
      "ts":      "2026-02-19T08:00:00Z",
      "next_ts": "2026-02-19T08:00:05Z",
      "values":  { ... },
      "seq":     101
    },
    {
      "ts":      "2026-02-19T08:00:05Z",
      "next_ts": "2026-02-19T08:00:10Z",
      "values":  { ... },
      "seq":     102
    }
  ],
  "complete": false
}
```

### Fields

| Field | Type | Description |
|---|---|---|
| `ts` | ISO 8601 timestamp | Original recording time on the device |
| `next_ts` | ISO 8601 timestamp or null | Timestamp of the next event in the buffer. `null` on the last entry when no subsequent event exists yet. |
| `values` | object | Status attributes, identical in schema to live channel output |
| `seq` | integer | Original sequence number from the status channel |
| `complete` | boolean | `true` on the final message in the response |
| `beginning` | boolean | `true` on the first message if the first entry sent is the oldest entry in the node's buffer — there is no earlier data (optional, included only when true) |
| `end` | boolean | `true` on the final message if the last entry sent is the newest entry in the node's buffer — there is no later data (optional, included only when true) |
| `entries` | array | Optional. Array of event objects, each containing `ts`, `next_ts`, `values`, and `seq` |

When `entries` is present, `complete`, `beginning`, and `end` apply to the message as
a whole, not to individual entries.

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
