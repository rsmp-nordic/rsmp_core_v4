---
title: Result
parent: Messages
nav_order: 3
permalink: /messages/result/
---

## Command Result
A command result is published after handling a command.

```
<node>/result/<code>[/<component>]
```

Examples:
```
dk/cph/45fe/result/tlc.plan.set      # TLC plan.set result (for main component) from node 45fe
dk/cph/45fe/result/sensor.17/dl/3    # Sensor M0017 for detector logic 3 from node 45fe
```
