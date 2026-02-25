---
title: Messages
permalink: /messages/
nav_order: 100
has_children: true
has_toc: false
---

# Messages
RSMP 4 defines the following messages with coresponding topic paths structures:


| Message type | Topic path |
|-|-|
| [Presence](presence.md) | `<node>/presence` |
| [Status](status.md) | `<node>/status/<code>[/<channel>][/<component>]` |
| [Channel](channel.md) | `<node>/channel/<code>/<channel>` |
| [Throttle](throttle.md) | `<node>/throttle/<code>/<channel>` |
| [Replay](replay.md) | `<node>/replay/<code>/<channel>[/<component>]` |
| [Fetch](fetch.md) | `<node>/fetch/<code>/<channel>[/<component>]` |
| [History](history.md) | `<supervisor>/history/<code>/<channel>[/<component>]` |
| [Command](command.md) | `<node>/command/<code>[/<component>]` |
| [Result](result.md) | `<node>/result/<code>[/<component>]` |
| [Alarm](alarm.md) | `<node>/alarm/<code>[/<component>]` |


Most topic paths (except presence and channel state) follow this layout:

```
<node>/<type>/<code>[/<part>]
```

- node: The unique identity of the RSMP node.
- type: the type of message, e.g. status, command, alarm.
- code: the code of command/status/alarm within the module.
- part: indentifies one or more [components](components.md) (or in the case of a channel message a [channel](channels.md))

The part is kept at the end of the topic path to ensure that you can retain status, commands and alarms for individual component.

The component can be left out as a shortcut to refer to the entire node.


## Payload CBOR Encoding
Message payloads consist of JSON encoded in binary format using [CBOR (Concise Binary Object Representation)](https://cbor.io).

Using CBOR encoding saves space, while still retaining the JSON data model, which is very easy to work with.

CBOR is natively supported in many MQTT tools, making it painless to interact with RMSP 4 messages encoded with CBOR. 
