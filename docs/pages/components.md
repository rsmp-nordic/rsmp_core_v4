---
layout: page
title: Components
nav_order: 5
---

# Components
A node has one or more components, representing physical of logical elements.

Components are referenced using component ids, which must be unique per node.

A component id consists of one or more levels, separated by slashes. For example, a traffic light controller (tc) with two signal groups (sg) and four detector logics (dl) might have these component ids:

```
tc
sg/1
sg/2
dl/video
dl/video/1
dl/video/2
dl/radar
dl/radar/1
dl/radar/2
dl/radar/3
dl/radar/4
```

Whitespace and non-printable characters are not allowed in component ids.


Unlike RSMP 3, RSMP 4 component ids does not include the node id. RMSP 3 components which include site ids, like `KK+AG0503=001DL001`, should be convert to RMSP 4 component ids, in this case `dl/1`.

## Component Types
Each component has a specific component type, like signal group or detector logic.

Modules typically work with specific component types. For example, the traffic light controller module  work with signal groups and detector logic.

A module can define messages that work with arbitrary component types. For example, a generic `system` module might work with components of any type to ftch the name and configuration of any component or to list components.

## Component Groups
You can address groups of component using intermediate levels. For example:

```
sg        # All signal groups
dl        # All detector logics (both video and radar)
dl/video  # All video detectors
dl/rader  # All radar detectors
```

Lists of specific components can be addressed with a comma-separated list:

```
dl/video/1,2   # Video detectors 1 and 2
```

You can also list groups:

```
sg,dl       # All signal groups and all detector logics
sg,dl/1     # The first signal groups and the first detector logic
```

Hyphen can be used for ranges of components:

```
dl/radar/2-4      # Radar detector from to 2 to 4, ie. 2, 3 and 4
```

You can combine commas and hyphens
```
dl/radar/2-6,15-19,25
```

Whitespace is not allowed before or after commas and hyphens.

## Component ids and Topic Paths
Component ids are used in topic paths, for example:

```
45fe/alarm/tlc.301/dl/6         # A0301 error for component dl.6 on node 45fe
45fe/command/sensor.17/sg/1     # Command M0017 to signal group 1 on node 45fe
```

RSMP 4 is based on MQTT which allow the last payload published to each topic path to be retained.
In the example above, this means that the alarms for `dl/6` and `dl/7` will be retained.
A node subscribing to these topics, or reconnecting after a network dropout, will then receive the latest alarm for each component.
This is useful to make sure that a newly connected node will receive the latest status immediately.

To ensure that the latest status for each component can be retained, groups or lists or components should not be used when sending alarms or statuses. Instead a status/alarm should be published for each component, using their individual component paths.

On the other hand, commands can use component groups or lists, because commands should not be retained. Instead QoS (Quality of Service) 1 or 2 should be used when sending commands to guarantee delivery. If the receiver is offline, the messages will be kept on the broker. When the receiver comes online again, they will be delivered. Since commands are not retained, newly connected nodes will not received previously send commands.

## Main component
Exactly one root level component must be configured as the main component for each node. This component is used to refer to the device as a whole.

```
tc     # configured as root component
sg/1
sg/2
dl
```

You can refer to the main component by using the component ID explicitly, or by omitting the component ID from the topic path. For example, assuming the component `tc` is configured as the main component, these two commands achieve the same:

```
45fe/command/tlc.plan.set/tc   # Send command to component tc
45fe/command/tlc.plan.set      # Shorthand to send command to the main component
```

The shorthand means you can address the main component without knowing its component id. This can be important when you want to inspect a new or unknown device where you don't know the id of the main component.

## Example: Traffic Light Controller
A traffic light controller managing a single intersection has a traffic controller component, representing the device as a whole, here called `tc`:

```
tc        # traffic controller, configured as main component
in        # intersection
sg/1      # signal group
sg/1      # signal group
```
Since there's only one intersection, it's not necessary to nest signal group components under the intersection, although you could.

A traffic light controller managing multiple intersections also has one traffic controller component, representing the device as a whole.
But it has several intersection components, under which signal groups can be nested:

```
tc         # traffic controller, configured as main component
in/1       # intersection
in/1/sg/1  # signal group
in/1/sg/2  # signal group
in/2       # intersection
in/2/sg/1  # signal group
in/2/sg/2  # signal group
```

With nesting, component ids for signal groups are slightly longer (e.g. `in/1/sg/1` vs `sg/1`), but on the other hand it's easy to see which intersection a signal group belongs to, you can reuse the same signal group names in several intersections and you have the possibility to easily address all signal groups in an intersection (e.g. `in/1/sg`).


