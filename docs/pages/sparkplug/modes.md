## Control Mode Metrics

Metrics related controller modes.

| Metric                                  | Description                  |
|-----------------------------------------|------------------------------|
| **/tlc/tc/1/modes**             			  | Controller is switched on    |
| **/tlc/tc/1/modes/sources**   		      | Controller activation source |

### Modes
The metric **/tlc/tc/1/modes** provides a list of active modes including:
- active
- manual control
- fixed time control
- isolated mode
- yellow flash
- all red
- police key

The metric **/tlc/tc/1/modes/source** indicate the soource of mode activations, e.g. reboot, operator, fault, etc.
