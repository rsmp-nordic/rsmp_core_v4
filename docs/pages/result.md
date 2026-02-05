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
dk/cph/45fe/result/tlc.2             # TLC M0002 (for main component) result from node 45fe
dk/cph/45fe/result/sensor.17/dl/3    # Sensor M0017 for detector logic 3 from node 45fe
```
