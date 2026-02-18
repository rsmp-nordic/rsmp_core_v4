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
| [Status](status.md) | `<node>/status/<code>[/<stream>][/<component>]` |
| [Stream](stream.md) | `<node>/stream/<code>/<stream>` |
| [Throttle](throttle.md) | `<node>/throttle/<code>/<stream>` |
| [Command](command.md) | `<node>/command/<code>[/<component>]` |
| [Result](result.md) | `<node>/result/<code>[/<component>]` |
| [Alarm](alarm.md) | `<node>/alarm/<code>[/<component>]` |



- node: The unique identity of the RSMP node, Can consist of one or more levels, but use must be consistent across a setup.
- type: the type of message, e.g. status, command, alarm.
- code: the code of command/status/alarm within the module.
- stream: stream related to the status code
- component: indentifies one or more [components](components.md).

The component is kept at the end of the topic path to ensure that you can retain status, commands and alarms for each component and easily subscribe to sub-trees.

The component can be left out as a shortcut to refer to the entire node.


## Payload CBOR Encoding
Message payloads consist of JSON encoded in binary format using [CBOR (Concise Binary Object Representation)](https://cbor.io).

Using CBOR encoding saves space, while still retaining the JSON data model, which is very easy to work with.

CBOR is natively supported in many MQTT tools, making it painless to interact with RMSP 4 messages encoded with CBOR. 
