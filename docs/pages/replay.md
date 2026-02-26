---
title: Replay
parent: Messages
nav_order: 8
permalink: /messages/replay/
---

# Replay
When a node reconnects after being offline, it MAY replay buffered status data
it collected while disconnected. Replay data is published to dedicated replay
topics, separate from live status topics.

```
<node>/replay/<code>/<channel>
```

Examples:
```
45fe/replay/tlc.groups/hourly           # replay of hourly signal group data
45fe/replay/traffic.count/hourly        # replay of hourly traffic data
```

## Purpose

While a node is offline, channels stop publishing to the broker. If the node
buffers status data locally during the outage, it can replay that data to
consumers once reconnected.

Replay covers the gap between the node going offline and coming back online.
Supervisors that were online during the outage will receive the data. Supervisors
that were themselves offline during the outage and missed the replay must retrieve
missing data via a [Fetch](fetch.md) request.

## Channel State During Replay

Channels operate normally during replay. The channel state MUST NOT change to
reflect replay activity. Live status data continues to flow on the normal status
topic path and MAY be interleaved with replay traffic.

## Replay Flow

On reconnect, the node:

1. Publishes `presence: online`.
2. Resumes all configured channels immediately — live data flows on normal status topics.
3. Begins replaying buffered data on replay topics in parallel, rate-limited to
   avoid starving live channels.
4. After all buffered data for a channel has been sent, publishes a final replay
   message with `done: true`. If there is no buffered data to replay, publishes
   `{"done": true}` immediately.

```
45fe/presence                          →  online
45fe/status/tlc.groups/live            →  (live — seq=151, current state)
45fe/replay/tlc.groups/hourly          →  (buffered t=08:00, seq=101)
45fe/replay/tlc.groups/hourly          →  (buffered t=09:00, seq=102)
45fe/status/tlc.groups/live            →  (live — seq=152, interleaved)
45fe/replay/tlc.groups/hourly          →  (buffered t=10:00, seq=103, done=true)
```

The live channel and the replay share the same sequence counter. A consumer
that was online before the outage knows the last live `seq` it received
(e.g. 100) and can immediately see where replay fits.

The `done: true` message signals that the node has finished replaying all
buffered data for that channel. Supervisors SHOULD wait for `done: true` before
deciding whether to send a [Fetch](fetch.md) request to fill any remaining gaps.

## Scope

A channel can be configured to buffer and replay its data, including send-on-change
data, or not. Replay is most appropriate for aggregated or periodic channels
(e.g. hourly traffic counts) where gaps break time-series continuity.
High-frequency send-on-change channels (e.g. signal group changes) SHOULD NOT
be configured for replay — the volume would be prohibitive and consumers
generally do not require complete live history.

## Payload

Replay messages use the same attribute schema as the channel's normal status
messages, with additional metadata fields.

Each MQTT message carries an `entries` array containing zero or more events.
The array MUST always be present; it MAY be empty.

```json
{
  "entries": [
    {
      "ts":      "2026-02-19T08:00:00Z",
      "next_ts": "2026-02-19T09:00:00Z",
      "values":  { ... },
      "seq":     101
    },
    {
      "ts":      "2026-02-19T09:00:00Z",
      "next_ts": "2026-02-19T10:00:00Z",
      "values":  { ... },
      "seq":     102
    }
  ],
  "done": true
}
```

Final message with a single entry:

```json
{
  "entries": [
    {
      "ts":      "2026-02-19T08:15:06Z",
      "next_ts": null,
      "values":  { ... },
      "seq":     3
    }
  ],
  "done": true
}
```

### Empty Replay Payload

When there is no buffered data to replay for a channel, the node MUST still
publish a single message to signal completion:

```json
{"entries": [], "done": true}
```

### Fields

| Field | Type | Description |
|---|---|---|
| `entries` | array | Array of event objects. MUST be present; MAY be empty. Each entry contains `ts`, `next_ts`, `values`, and `seq`. |
| `ts` | ISO 8601 timestamp | Original recording time on the device |
| `next_ts` | ISO 8601 timestamp or null | Timestamp of the next event in the buffer. `null` on the last entry when no subsequent event exists yet. |
| `values` | object | Status attributes, identical in schema to live channel output |
| `seq` | integer | Original sequence number from the status channel — continuous with live messages |
| `done` | boolean | `true` on the final message in the replay sequence. Omitted on non-final messages. The supervisor MUST wait for this before deciding whether to fetch missing data. |
| `beginning` | boolean | `true` on the first message if the first entry sent is the oldest entry in the node's buffer — there is no earlier data (optional, included only when true) |

`done` and `beginning` apply to the message as a whole, not to individual entries.

Consumers MUST use `ts` to position replay messages in the correct place in
the time series, not the MQTT message arrival time.

Consumers MAY detect dropped replay messages by checking for gaps in `seq`.

## Interrupted Replay

If a node goes offline again during replay, the replay is abandoned without
sending `done: true`. On the next reconnect the node resumes replay from
where it left off — it MUST NOT re-send messages it has already replayed.
Sequence numbers continue forward uninterrupted.

## MQTT Behavior

- QoS: `1`
- Retain: `false`

## Subscribing

Supervisors subscribe to replay topics the same way as status topics:

```
45fe/replay/#                  # all replay data from node 45fe
+/replay/tlc.groups/hourly     # hourly signal group replay from any node
```

## Rate Limiting

A node MUST rate-limit replay publishing. Replay rate is a per-channel
configuration setting. Rate limiting prevents replay traffic from delaying
live channel publications.
