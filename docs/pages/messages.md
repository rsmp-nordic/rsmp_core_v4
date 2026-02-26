---
title: Messages
permalink: /messages/
nav_order: 100
has_children: true
has_toc: false
---

# Messages
RSMP 4 defines the following messages with corresponding topic path structures:


| Message type | Topic path |
|-|-|
| [Presence](presence.md) | `<node>/presence` |
| [Status](status.md) | `<node>/status/<code>[/<channel>]` |
| [Channel](channel.md) | `<node>/channel/<code>[/<channel>]` |
| [Throttle](throttle.md) | `<node>/throttle/<code>[/<channel>]` |
| [Replay](replay.md) | `<node>/replay/<code>/<channel>` |
| [Fetch](fetch.md) | `<node>/fetch/<code>[/<channel>]` |
| [History](history.md) | `<supervisor>/history/<code>[/<channel>]` |
| [Command](command.md) | `<node>/command/<code>` |
| [Result](result.md) | `<node>/result/<code>` |
| [Alarm](alarm.md) | `<node>/alarm/<code>` |


Most topic paths (except presence and channel state) follow this layout:

```
<node>/<type>/<code>
```

- node: The unique identity of the RSMP node.
- type: the type of message, e.g. status, command, alarm.
- code: the code of command/status/alarm within the module.

Components are not part of the topic path. When a message relates to specific
[components](components.md), the component is identified in the message payload.
This keeps topics simple, avoids parsing ambiguity, and allows atomic multi-component
messages.


## Payload CBOR Encoding
Message payloads consist of JSON encoded in binary format using [CBOR (Concise Binary Object Representation)](https://cbor.io).

Using CBOR encoding saves space, while still retaining the JSON data model, which is very easy to work with.

CBOR is natively supported in many MQTT tools, making it painless to interact with RMSP 4 messages encoded with CBOR. 
