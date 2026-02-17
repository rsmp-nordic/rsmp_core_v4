---
title: Stream
parent: Messages
nav_order: 6
permalink: /messages/stream/
---

# Stream
Stream messages publish lifecycle/state information for configured streams.
They do not carry status data values.

```
<node>/stream/<code>/<stream>
```

Examples:
```
dk/cph/45fe/stream/tlc.groups/live      # state for live stream of signal groups
dk/cph/45fe/stream/tlc.groups/hourly    # state for hourly stream of signal groups
```

Payload (CBOR encoded JSON) is explicitly defined as:

```json
{"state": "running" | "stopped"}
```

Rules:
- The payload MUST include exactly one key: `state`.
- `state` MUST be one of: `running`, `stopped`.
- No additional payload keys are currently defined for this topic.

## MQTT Behavior
- QoS: `1`
- Retain: `true`

Each stream topic keeps the latest known state as a retained message, so
supervisors can recover stream state immediately after subscribing.

Runtime control is performed via [Throttle](throttle.md) messages.

## When to Publish
A node publishes stream state:
- when a stream starts
- when a stream stops
- when stream configuration changes affect effective runtime state
- when the node connects (republish current state for all streams)
