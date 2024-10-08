---
layout: page
title: Presence
parent: Message Types
nav_order: 0
permalink: /messages/presence/
---

# Presence
Supervisors are informed whenever nodes come online or go offline.

```
presence/<sender>
````

Examples:
```
presence/45fe   # node 45fe went online/offline
```

## Connect
After connecting, the node publishes to the `state/<id>` topic, with a payload that indicates it's online: 

```mermaid
 graph LR;
      A[Device 65a3]-->|state/65a3: ok| Broker;
      Broker-->|state/65a3: hello| Supervisor;
```

## Disconnect
A graceful disconnect (e.g. if the device chooses to power down to to a low battery, or a manual shutdown) can be handled by the device posting to the 'state' topic:
`state/<id>`

```mermaid
 graph LR;
      A[Device 65a3]-->|state/65a3: shutdown| Broker
      Broker-->|state/65a3: bye| Supervisor;
```

An unexpected disconnect is handled using Last Will. When connecting to the broker, the site set Last Will and Testament (LWT), which will be published by the broker on behalf of the site, in case he site is disconnected. The topic `state/<id>` is used, with a payload indicating that the site was disconnected unexpectedly:

```mermaid
 graph LR;
      A[Device 65a3]
      Broker-->|state/65a3: gone| Supervisor;
```

More about Last Will:
https://www.hivemq.com/blog/mqtt-essentials-part-9-last-will-and-testament/

## Disconnected Supervisor
Retained message should be used when publishing to `state/<id>`. If the supervisor is disconnected from the broker, the broker will retain the last messages send be each device. When the supervisor comes back online, it will immediately get the current state of all nodes.
