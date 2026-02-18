---
title: Result
parent: Messages
nav_order: 3
permalink: /messages/result/
---

## Command Result
```
<node>/result/<code>[/<component>]
```

Send after handling a command.

When a command is received the Result Topic indicates the topic to publish the result to. The `node` is the id of the node that send the command.


Examples:
```
45fe/result/tlc.plan.set      # TLC plan.set result from node 45fe
45fe/result/sensor.17/dl/3    # Sensor M0017 for detector logic 3 from node 45fe
```
