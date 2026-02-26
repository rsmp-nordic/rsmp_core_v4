---
layout: page
title: Components
nav_order: 5
---

# Components
A node has one or more components, representing physical or logical parts of
the node.

Components are referenced using component ids, which MUST be unique per node.

A component id consists of one or more levels, separated by slashes. For
example, a traffic light controller with two signal groups (sg) and four
detector logics (dl) might have these component ids:

```
sg/1
sg/2
dl/video/1
dl/video/2
dl/radar/1
dl/radar/2
dl/radar/3
dl/radar/4
```

Whitespace and non-printable characters are not allowed in component ids.

Unlike RSMP 3, RSMP 4 component ids do not include the node id. RSMP 3
components which include site ids, like `KK+AG0503=001DL001`, should be
converted to RSMP 4 component ids, in this case `dl/1`.

## Component Types
Each component has a specific component type, like signal group or detector
logic.

Modules typically work with specific component types. For example, the traffic
light controller module works with signal groups and detector logic.

A module can define messages that work with arbitrary component types. For
example, a generic `system` module might work with components of any type to
fetch the name and configuration of any component or to list components.

## Components in Payloads
Components are **not** part of the MQTT topic path. When a message relates to
specific components, the component is identified in the message payload.

This keeps the topic structure simple and unambiguous:

```
<node>/status/<code>[/<channel>]
<node>/alarm/<code>
<node>/command/<code>
```

The component appears in the payload where needed. For example, a status
payload for signal groups includes per-component data in `values`:

```json
{
  "entries": [
    {
      "ts": "2026-02-24T10:00:00.000Z",
      "values": {
        "signalgroupstatus": {"sg/1": "G", "sg/2": "r"},
        "cyclecounter": 42
      },
      "seq": 123
    }
  ]
}
```

A command targeting a specific component includes it in the payload:

```json
{
  "component": "dl/radar/1",
  "values": {"action": "reset"}
}
```

This design means that one MQTT message can carry data for multiple components
atomically, and periodic full updates serve as retained snapshots for new
subscribers.

## Component Groups
You can address groups of components using intermediate levels. For example:

```
sg        # All signal groups
dl        # All detector logics (both video and radar)
dl/video  # All video detectors
dl/radar  # All radar detectors
```

Lists of specific components can be addressed with a comma-separated list:

```
dl/video/1,2   # Video detectors 1 and 2
```

You can also list groups:

```
sg,dl       # All signal groups and all detector logics
```

Hyphen can be used for ranges of components:

```
dl/radar/2-4      # Radar detectors 2, 3 and 4
```

You can combine commas and hyphens:

```
dl/radar/2-6,15-19,25
```

Whitespace is not allowed before or after commas and hyphens.

Component groups and lists are useful in command payloads when addressing
multiple components at once.

## Main Component
When a message applies to the node as a whole rather than a specific
component, the `component` field is omitted from the payload.

## Example: Traffic Light Controller
A traffic light controller managing a single intersection might have:

```
sg/1      # signal group
sg/2      # signal group
```

Since there is only one intersection, it is not necessary to nest signal group
components under an intersection, although you could.

A traffic light controller managing multiple intersections has several
intersection components, under which signal groups can be nested:

```
in/1       # intersection
in/1/sg/1  # signal group
in/1/sg/2  # signal group
in/2       # intersection
in/2/sg/1  # signal group
in/2/sg/2  # signal group
```

With nesting, component ids for signal groups are slightly longer
(e.g. `in/1/sg/1` vs `sg/1`), but it is easy to see which intersection a
signal group belongs to, you can reuse signal group names across intersections,
and you can address all signal groups in an intersection (e.g. `in/1/sg`).
