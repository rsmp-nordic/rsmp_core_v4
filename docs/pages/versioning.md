---
title: Versioning
nav_order: 7
permalink: /versioning/
---

# Versioning

## MQTT
Nodes and supervisor must use compatible version of MQTT. This is handled by the MQTT protocol and the broker.

## RSMP 4 Topic Evolution
The stream state topic family `<node>/stream/<code>/<stream>` is an additive
extension. Existing stream data topics under `<node>/status/...` remain
canonical and unchanged, so existing status consumers remain compatible.
