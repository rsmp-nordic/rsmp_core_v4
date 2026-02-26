---
title: Alarm
parent: Messages
nav_order: 4
permalink: /messages/alarm/
---

# Alarm
A node sends alarm messages to inform consumers about alarm state changes.

```
<node>/alarm/<code>
```

Examples:
```
45fe/alarm/tlc.hardware.error   # serious hardware error on node 45fe
45fe/alarm/tlc.detector         # detector error on node 45fe
```

When an alarm relates to a specific component, the component is identified in
the payload, not in the topic path. This means all alarm events for a given
code share a single topic, regardless of which component is affected.

## Payload

Alarm messages use the same `entries` array structure as status messages. Each
entry records one alarm event with a precise timestamp.

A single-event message has one entry; when multiple alarm events occur within
a short period they MAY be batched into a single message with multiple entries.

### Event Update

An event update is published when an alarm becomes active or inactive on any
component. It contains only the changed alarm(s).

```json
{
  "entries": [
    {
      "ts": "2026-02-24T10:02:30.000Z",
      "component": "dl/7",
      "active": true,
      "values": {"message": "No response from detector"}
    }
  ]
}
```

When the alarm applies to the node as a whole (no specific component), the
`component` field is omitted:

```json
{
  "entries": [
    {
      "ts": "2026-02-24T10:05:00.000Z",
      "active": true,
      "values": {"message": "Critical hardware failure"}
    }
  ]
}
```

Batched example — two detector alarms occurring close together:

```json
{
  "entries": [
    {
      "ts": "2026-02-24T10:02:30.000Z",
      "component": "dl/1",
      "active": true,
      "values": {"message": "No response from detector"}
    },
    {
      "ts": "2026-02-24T10:02:31.200Z",
      "component": "dl/7",
      "active": true,
      "values": {"message": "No response from detector"}
    }
  ]
}
```

### Full Update

A full update contains the complete alarm state for the code across all
components. It is published:
- When the node first connects
- Periodically (e.g. every 5 minutes) as a heartbeat

Full updates are published with `retain = true` so new subscribers
immediately receive the current alarm state.

```json
{
  "entries": [
    {
      "ts": "2026-02-24T10:02:30.000Z",
      "component": "dl/1",
      "active": true,
      "values": {"message": "No response from detector"}
    },
    {
      "ts": "2026-02-24T10:02:31.200Z",
      "component": "dl/7",
      "active": true,
      "values": {"message": "No response from detector"}
    }
  ],
  "full": true
}
```

When no alarms are active, the full update has an empty entries array:

```json
{
  "entries": [],
  "full": true
}
```

### Fields

| Field | Type | Description |
|---|---|---|
| `entries` | array | One or more alarm event objects |
| `ts` | ISO 8601 timestamp | When the alarm state changed |
| `component` | string | Component id (omitted when alarm applies to the node as a whole) |
| `active` | boolean | `true` if the alarm is active, `false` if cleared |
| `values` | object | Alarm attributes (e.g. message, severity) |
| `full` | boolean | `true` on full updates. Omitted on event updates. |

## MQTT Behavior

| Property | Value |
|---|---|
| QoS | `1` |
| Retain | `true` for full updates, `false` for event updates |

## Subscription Patterns

```
45fe/alarm/#            # all alarms from node 45fe
+/alarm/#               # all alarms from all nodes
45fe/alarm/tlc.detector # detector alarms from node 45fe
```
