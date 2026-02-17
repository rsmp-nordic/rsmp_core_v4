---
title: Throttle
parent: Messages
nav_order: 7
permalink: /messages/throttle/
---

# Throttle
Throttle messages control stream runtime state (start/stop) for a specific
stream on a node.

```
<node>/throttle/<code>/<stream>
```

Examples:
```
dk/cph/45fe/throttle/tlc.groups/live      # start/stop live signal group stream
dk/cph/45fe/throttle/traffic.volume/5s    # start/stop 5s traffic stream
```

For single-stream statuses, `default` can be used as stream name:

```
dk/cph/45fe/throttle/tlc.plan/default
```

Payload (CBOR encoded JSON):

```json
{"action": "start" | "stop"}
```

Rules:
- The payload MUST include exactly one key: `action`.
- `action` MUST be one of: `start`, `stop`.
- No additional payload keys are currently defined for this topic.

## MQTT Behavior
- QoS: `1`
- Retain: `false`

Throttle commands trigger stream state updates on corresponding stream topics:

```
<node>/stream/<code>/<stream>
```
