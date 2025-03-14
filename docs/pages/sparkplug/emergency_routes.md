# Emergency Route Metrics

| Metric                                   | Description                                  |
|------------------------------------------|----------------------------------------------|
| **tlc/tc/1/emergency_route/list**        | Available emergency routes                   |
| **tlc/tc/1/emergency_route**             | Current emergency route                      |

## Usage
The metric `tlc/tc/1/emergency_route/available` list all configured emergency routes.

The metric `tlc/tc/1/emergency_route` indicates the currently active emergency route, and associated metadata like the source of the current activation.
Change the metric to change or clear the current emergency route.