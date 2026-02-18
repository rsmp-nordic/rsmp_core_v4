---
title: Aggregated Status
parent: Messages
nav_order: 6
permalink: /messages/aggregate/
---
## Aggregated Status
We would perhaps consider this a status like any other, just defined to indicate the overall status of all components. It would be linked to the main component. Eg:

`<node>/status/system.aggregated`

An option would also be to split each aggregated status part into topics:
`<node>/status/system.aggregated.high/main`
`<node>/status/system.aggregated.low/main`
`<node>/status/system.aggregated.local_mode/main`