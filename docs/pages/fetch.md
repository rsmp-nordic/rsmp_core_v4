---
title: Fetch
parent: Messages
nav_order: 9
permalink: /messages/fetch/
---

# Fetch
A supervisor sends a fetch message to request status data for a specific time
range from a node. The node responds with [History](history.md) messages
published directly to the requesting supervisor.

```
<node>/fetch/<code>/<channel>[/<component>]
```

If the channel name is omitted in the corresponding status topic, it MUST also be omitted here:

```
<node>/fetch/<code>
```

Examples:
```
45fe/fetch/tlc.groups/hourly              # fetch hourly signal group data from node 45fe
45fe/fetch/tlc.plan                       # fetch current plan (single channel, name omitted)
45fe/fetch/traffic.count/hourly/dl/1      # fetch hourly traffic data for detector logic 1
```

## Purpose

Fetch allows a supervisor to retrieve status data it missed — for example
because it was itself offline, because data was lost in transit, or because it
came online after an auto-replay had already completed.

Unlike [Replay](replay.md), which is producer-initiated and broadcasts to all
connected supervisors, fetch is consumer-initiated and routes the response to
the requesting supervisor only.

## MQTT Properties

Fetch uses MQTT 5 Request/Response:

- The supervisor MUST set the **Response Topic** to `<supervisor>/history/<code>/<channel>[/<component>]`,
  where `code`, `channel`, and `component` match those in the fetch topic path. This strict format is mandated for global observability and debugging by third-party clients.
- The supervisor MUST set **Correlation Data** to a unique identifier for the
  request. The node MUST echo this value in every history message it publishes
  in response.
- The node treats the Response Topic as an opaque return address and publishes
  to it verbatim without validation.

| Property | Value |
|---|---|
| QoS | `1` |
| Retain | `false` |
| Response Topic | `<supervisor>/history/<code>/<channel>[/<component>]` |
| Correlation Data | Set by supervisor |
| Message Expiry Interval | SHOULD be set — avoids stale fetches being delivered if the node was offline |

## Payload

The payload is CBOR encoded (represented here as JSON). Timestamps MUST be encoded as ISO 8601 strings.

```json
{
  "from": "2026-02-19T08:00:00Z",
  "to":   "2026-02-19T10:00:00Z"
}
```

| Field | Type | Description |
|---|---|---|
| `from` | ISO 8601 timestamp | Start of requested time range (inclusive) |
| `to` | ISO 8601 timestamp | End of requested time range (exclusive) |

## Fetching Deltas

When a supervisor fetches a time window, it might receive only delta updates if no full update occurred in that window. The node returns whatever is in its buffer for the requested range. The supervisor is responsible for fetching further back in time to find a full update if it needs to establish a baseline state.

## Stopped Channels

Fetch requests are valid even if the target channel is currently in a `stopped` state. The node will return any buffered data that falls within the requested time range.

## Node Subscription

Nodes MUST subscribe to their own fetch topic on connect:

```
<node>/fetch/#
```

## Relationship to Replay

Fetch and replay are complementary:

| | Replay | Fetch |
|---|---|---|
| Initiated by | Producer (node) | Consumer (supervisor) |
| Triggered | Automatically on reconnect | Explicitly by supervisor |
| Recipients | All connected supervisors | Requesting supervisor only |
| Use case | Fill gap after device outage | Fill gap after supervisor outage or data loss |
