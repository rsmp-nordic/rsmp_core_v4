---
title: Command
parent: Messages
nav_order: 2
permalink: /messages/command/
---

## Command
```
<node>/command/<code>[/<component>]
```

Examples:
```
45fe/command/tlc.plan.set                # set signal plan on node 45fe
45fe/command/sensor.reset/radar/1       # reset detector radar/1 on node 45fe
```

The payload contains the parameters.

## Result
The result is returned to the supervisor by publishing to the Response Topics set in the message.
The response topic must include the id of the supervisor sending the command.

For example, when a supervisor with id 22ba sends the command `45fe/command/tlc.plan.set`, the response topic must be be set to to `22ba/result/tlc.plan.set`. The site published the result to this topic.

