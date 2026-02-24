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

If the channel name is omitted in the corresponding status topic, it MUST also be omitted here:

```
<node>/channel/<code>
```

Channel state applies to the entire channel across all components. Therefore, the `[/<component>]` segment is never present in the channel topic path.

Examples:
```
45fe/channel/tlc.groups/live      # state for live channel of signal groups
45fe/channel/tlc.groups/hourly    # state for hourly channel of signal groups
45fe/channel/tlc.plan             # state for current plan (single channel, name omitted)
```

Payload (CBOR, represented here as JSON) is explicitly defined as:

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
reconnect, including event data. When replay is enabled, the node
buffers data during outages and publishes it to [Replay](replay.md) topics
after reconnecting.

Replay is off by default. A maximum buffer duration can also be configured to
limit how much data the node retains.

## Batching
A channel can be configured with a **batch interval** to group multiple
updates into a single MQTT message. Instead of publishing each event
immediately, the node accumulates events in memory and publishes them
together as an `entries` array at the configured interval.

Batching reduces per-message MQTT overhead on constrained networks while
preserving the exact timestamp and sequence number of each individual event.

Batching is optional and orthogonal to other channel settings. Channels that
require minimal latency (e.g. live signal groups) should not use batching.
Channels with high event rates on constrained links (e.g. periodic sensor
readings, traffic counts) benefit most from batching.

See [Status](status.md) for payload format details and retention rules
for batched messages.

## Timeout
A channel can be started with an optional timeout. When the timeout expires,
the channel stops automatically and retained data is cleared from the broker.

This is useful for channels that consume significant bandwidth. A consumer that
wants to keep the channel running MUST periodically restart it before the timeout
expires.

See [Throttle](throttle.md) for how to specify a timeout when starting a channel.
