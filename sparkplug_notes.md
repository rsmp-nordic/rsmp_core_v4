

# SparkPlug

## Basics
Topics have the form:

```
namespace/group_id/message_type/edge_node_id/[device_id]
```

While metrics can have any form:
```
a/b/c...
```

The topic can be used to route the mesage to the right device, while metrics can define to structure data on that device.


## Metrics
A message to/from a device can contain multiple metrics. You can also send multiple values for the same metric, with different timestamps as well as historic metrics.

This is efficient, but also means you cannot retain data, since you don't have a topic path for each metric. Instead you need to define when/how data is resend, e.g. when applications (re)connect.

MQTT guarantees ordering of messages but only within each topic path.
Sequence numbers and reorder timeout are used to check ordering.


## Primary Host
Nodes/Device can be configured to use a primary host, but does not have to.

> If the Edge Node is configured to use a Primary Host Application, it must also watch for STATE messages from the Primary Host Application via an MQTT subscription. If the Primary Host Application denotes it is offline, the Edge Node must disconnect from the current MQTT server



## Limitations
No communication between nodes. Even though they are all connected to the same MQTT broker, SparkPlug only defines how hosts can communicate with devices. There is no support for communication betweeen devices or between hosts. It's possible to use the udnerlying MQTT connection to send other messages, but then it's not SparkPlug.

No way to subscribe to a single metric, as all metrics from a device is published to the same topic path.

No end-to-end encryption as the payload format is fixed.

Requirment for Google Protocol Buffer.



