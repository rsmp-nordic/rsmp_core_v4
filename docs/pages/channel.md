---
title: Channel
parent: Messages
nav_order: 6
permalink: /messages/channel/
---

# Channel
Channel messages publish lifecycle/state information for configured channels.
They do not carry status data values.

```
<node>/channel/<code>/<channel>
```

Examples:
```
45fe/channel/tlc.groups/live      # state for live channel of signal groups
45fe/channel/tlc.groups/hourly    # state for hourly channel of signal groups
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

Each channel topic keeps the latest known state as a retained message, so
supervisors can recover channel state immediately after subscribing.

Runtime control is performed via [Throttle](throttle.md) messages.

## When to Publish
A node publishes channel state:
- when a channel starts
- when a channel stops
- when channel configuration changes affect effective runtime state
- when the node connects (republish current state for all channels)

## Discovery
Supervisors discover available channels by subscribing to retained channel topics:

```
<node>/channel/#
```

Because channel state messages are retained, a subscriber immediately receives
the latest known state for each channel on connect.

Subscription patterns:
- `45fe/channel/#` — all channel states for a device
- `45fe/channel/tlc.groups/#` — all channel states for signal group status
- `+/channel/#` — all channel states from all devices

## Replay
A channel can be configured to buffer published data locally and replay it on
reconnect, including send-on-change data. When replay is enabled, the node
buffers data during outages and publishes it to [Replay](replay.md) topics
after reconnecting.

Replay is off by default. A maximum buffer duration can also be configured to
limit how much data the node retains.

## Pruning
Channels can be configured to automatically stop when consumers disappear, or
have been offline for a predefined period.

This is useful for channels that consume significant bandwidth. Automatic
stopping is based on the `presence` message, which informs the node when other
nodes go online/offline.

When all known consumers go offline, a prune timer starts. If no consumer
comes back within the prune timeout, the channel is stopped and retained data
is cleared from the broker.
