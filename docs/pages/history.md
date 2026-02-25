---
title: History
parent: Messages
nav_order: 10
permalink: /messages/history/
---

# History
History messages are published by a node in response to a [Fetch](fetch.md)
request. They are routed to the requesting supervisor via the
Response Topic set in the fetch message.

```
<supervisor>/history/<code>/<channel>[/<component>]
```

If only one channel is defined for the code, the channel name can be omitted:

```
<supervisor>/history/<code>
```

Examples:
```
22ba/history/tlc.groups/hourly              # history for signal groups, he hourly channel
22ba/history/tlc.plan                       # history for current plan (default channel)
22ba/history/traffic.count/hourly/dl/1      # history for traffic counts hourly channel, on component dl/1
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

Each MQTT message carries an `entries` array containing zero or more events.
The array MUST always be present; it MAY be empty.

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
| `entries` | array | Array of event objects. MUST be present; MAY be empty. Each entry contains `ts`, `next_ts`, `values`, and `seq`. |
| `ts` | ISO 8601 timestamp | Original recording time on the device |
| `next_ts` | ISO 8601 timestamp or null | Timestamp of the next event in the buffer. `null` on the last entry when no subsequent event exists yet. |
| `values` | object | Status attributes, identical in schema to live channel output |
| `seq` | integer | Original sequence number from the status channel |
| `complete` | boolean | `true` on the final message in the response |
| `beginning` | boolean | `true` on the first message if the first entry sent is the oldest entry in the node's buffer — there is no earlier data (optional, included only when true) |
| `end` | boolean | `true` on the final message if the last entry sent is the newest entry in the node's buffer — there is no later data (optional, included only when true) |

`complete`, `beginning`, and `end` apply to the message as a whole, not to individual entries.

`seq` uses the same counter as live and replay messages for the same channel,
allowing the supervisor to locate history messages precisely within the overall
data series.

Consumers MUST use `ts` to position history messages in the time series, not
the MQTT message arrival time.

## Empty Response

If no data is available for the requested range, the node MUST send a single
message with `complete: true` and an empty `entries` array:

```json
{
  "entries": [],
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
