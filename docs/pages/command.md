---
title: Command
parent: Messages
nav_order: 2
permalink: /messages/command/
---

## Command
```
<node>/command/<code>
```

Examples:
```
45fe/command/tlc.plan.set       # set signal plan on node 45fe
45fe/command/sensor.reset       # reset a detector on node 45fe
```

The payload contains the command parameters. When a command targets a specific
component, the component is included in the payload:

```json
{
  "component": "dl/radar/1",
  "values": {"action": "reset"}
}
```

When the command applies to the node as a whole, the `component` field is
omitted.

## Result
The result is returned to the supervisor by publishing to the Response Topic set in the message.
The response topic MUST include the id of the supervisor sending the command.

For example, when a supervisor with id 22ba sends the command `45fe/command/tlc.plan.set`, the response topic MUST be set to `22ba/result/tlc.plan.set`. The site publishes the result to this topic.

