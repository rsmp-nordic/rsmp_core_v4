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

Used to send a command to a node.

Examples:
```
45fe/command/tlc.plan.set              # set signal plan (for main component) on node 45fe
45fe/command/traffic.17/sg/1           # hypothetical M0017 set detector threshold for signal group 1 on node 45fe
```


All devices subscribe to a command topic that includes their own id, which the supervisor can publish to. 

Device subscribe to  `<node>/command/<code>[/<component>]`

node: the id of the device itself
code: the command code

The payload contains the parameters in JSON format.

For example, a traffic light with id 45fe might subscribe to all commands for all components using:
`45fe/command/#`

Now the supervisor can send commands to this particular traffic light and component by publishing to `45fe/command/...`

Suppose the traffic light has these components:

- main: the main controller, handles, eg. signal program changes
- sg1: signal group 1
- sg2: signal group 2
- dl1: detector logic 1
- dl2: detector logic 2

And let's suppose the traffic light supports these modules:
- tlc: traffic light controller commands
- detector: traffic detectors used for local traffic control


### Change signal plan
`45fe/command/tlc.plan.set`

We send the command to the `main` component (by omitting the component part).
We use `tlc.plan.set` command.

The payload would include the arguments, like what plan to switch to.

### Set detector sensibility
`45fe/command/detector.sensibility/dl/1`

We send the command to the `dl/1` component.
We use `detector.sensibility` command.

The payload would include the arguments, like the sensibility value, as well as a unique commmand tracking id.

### Sending to many devices
A traffic light with id 45fe could subscribe to a topic with the id 'all' (if supported by project governance):
`all/command/#`

The supervisor can publish to this topic to e.g. change the signal plan on all traffic lights at the same:
`all/command/tlc.plan.set`

## Result
When you send a command, you set a Result Topics which the node will use to publish a result message to.
For exampe, a supervisor sending a command could set the result topic:
`<supervisor>/result/<code>[/<component>]`.

The payload includes informationk about the node and the command tracking id.

When a supervisor 22ba changes the plan on node 45fe with `45fe/command/tlc.plan.set`, the response would be send to `22ba/result/tlc.plan.set`, so the supervisor know the result of the command.

A supervisor can receive result related to other supervisors by subscribe to e.g. `+/result/tlc.plan.set` or `+/result/#`.
