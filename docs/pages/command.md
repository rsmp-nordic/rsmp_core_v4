---
title: Command
parent: Messages
nav_order: 2
permalink: /messages/command/
---

## Command
```
<node_id>/command/<code>[/<component>]
```

Examples:
```
dk/cph/45fe/command/tlc.plan.set              # set signal plan (for main component) on node 45fe
dk/cph/45fe/command/traffic.17/sg/1           # hypothetical M0017 set detector threshold for signal group 1 on node 45fe
```


All devices subscribe to a command topic that includes their id, which the supervisor can publish to. 

Device subscribe to  `<node_id>/command/<code>[/<component>]`

node_id: the id of the device itself (incl. hierarchy)
code: the command code (flattened)

The payload contains the parameters in JSON format.

For example, a traffic light with id 45fe might subscribe to all commands for all components using:
`dk/cph/45fe/command/#`

Now the supervisor can send commands to this particular traffic light and component by publishing to `dk/cph/45fe/command/...`

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
`dk/cph/45fe/command/tlc.plan.set`

We send the command to the `main` component (by omitting the component part).
We use `plan.set` command in the `tlc` module (code `tlc.plan.set`).

The payload would include the arguments, like what plan to switch to.

### Set detector sensibility
`dk/cph/45fe/command/detector.sensibility/dl1`

We send the command to the `dl1` component.
We use `sensibility` command in the `detector` module (code `detector.sensibility`).

The payload would include the arguments, like the sensibility value.

### Sending to many devices
A traffic light with id 45fe could subscribe to a topic with the id 'all' (if supported by project governance):
`dk/cph/all/command/#`

The supervisor can publish to this topic to e.g. change the signal plan on all traffic lights at the same:
`dk/cph/all/command/tlc.plan.set`

## Result
How would command results be handled? We would use the request-result pattern, which is based on Result Topics. When you send a command, you pass a topic that would want to result to be published to. A supervisor sending a command could pass the result topic:
`<supervisor_id>/result/<code>[/<component>]`.

When a supervisor with id 22ba changing plan with `dk/cph/45fe/command/tlc.plan.set`, the response would be send to `dk/cph/22ba/result/tlc.plan.set`.

All supervisor can receive result if the just subscribe to `dk/cph/+/result/tlc.plan.set`.
