---
title: Alarm
parent: Messages
nav_order: 4
permalink: /messages/alarm/
---

# Alarm
```
<node>/alarm/<code>[/<component>]
```

Examples:
```
45fe/alarm/tlc.hardware.error   # serious hardware error (for main component) on node 45fe
45fe/alarm/tlc.301/dl/7    # A0301 serious detector error for component dl.7 on node 45fe
```

For example, a traffic light `bb35` that has a hardware error in detector logic 4 might publish to:
`bb35/alarm/tlc.hardware.error/dl/4`


## Subscribing
Supervisors can subscribe to all alarms from a specific device:
`45fe/alarm/#`

Or all alarms within a region (assuming fixed prefix depth):
`+/alarm/#`


