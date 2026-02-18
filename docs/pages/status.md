---
title: Status
parent: Messages
nav_order: 1
permalink: /messages/status/
---

## Status
```
<node>/status/<code>[/<stream>][/<component>]
```

When a status has only a single stream and no component segments, the stream
name may be omitted:

```
<node>/status/<code>
```

Examples:
```
45fe/status/tlc.groups/live       # live stream of signal group status
45fe/status/tlc.groups/hourly     # hourly aggregated signal group status
45fe/status/tlc.plan              # current plan (single stream, name omitted)
45fe/status/traffic.count/hourly/dl/1  # hourly traffic data for detector logic 1
```
