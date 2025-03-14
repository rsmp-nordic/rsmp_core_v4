## Control Mode Metrics

This document outlines metrics for various system control modes. Each mode includes a boolean flag and a corresponding source string.

| Metric                                  | Description                  |
|-----------------------------------------|------------------------------|
| **/tlc/tc/1/mode/active**               | Controller is switched on    |
| **/tlc/tc/1/mode/active/source**        | Controller activation source |
| **/tlc/tc/1/mode/manual**               | Manual control enabled       |
| **/tlc/tc/1/mode/manual/source**        | Manual control source        |
| **/tlc/tc/1/mode/fixed_time**           | Fixed time control active    |
| **/tlc/tc/1/mode/fixed_time/source**    | Fixed time control source    |
| **/tlc/tc/1/mode/isolated**             | Isolated mode active         |
| **/tlc/tc/1/mode/isolated/source**      | Isolated mode source         |
| **/tlc/tc/1/mode/yellow_flash**         | Yellow flash active          |
| **/tlc/tc/1/mode/yellow_flash/source**  | Yellow flash source          |
| **/tlc/tc/1/mode/all_red**              | All red active               |
| **/tlc/tc/1/mode/all_red/source**       | All red source               |
| **/tlc/tc/1/mode/police_key**           | Police key active            |
| **/tlc/tc/1/mode/police_key/source**    | Police key source            |

### Overview
Each control mode has:

- A boolean metric indicating if the mode is active.
- A source metric providing context for its activation.

### Modes

#### S0007 Controller Switched On
- **/tlc/tc/1/mode/active:** Controller is switched on.
- **/tlc/tc/1/mode/active/source:** Source of activation.

#### S0008 Manual Control
- **/tlc/tc/1/mode/manual:** Manual control enabled.
- **/tlc/tc/1/mode/manual/source:** Manual control source.

#### S0009 Fixed Time Control
- **/tlc/tc/1/mode/fixed_time:** Fixed time control active.
- **/tlc/tc/1/mode/fixed_time/source:** Fixed time control source.

#### S0010 Isolated Mode
- **/tlc/tc/1/mode/isolated:** Isolated mode active.
- **/tlc/tc/1/mode/isolated/source:** Isolated mode source.

#### S0011 Yellow Flash
- **/tlc/tc/1/mode/yellow_flash:** Yellow flash active.
- **/tlc/tc/1/mode/yellow_flash/source:** Yellow flash source.

#### S0012 All Red
- **/tlc/tc/1/mode/all_red:** All red active.
- **/tlc/tc/1/mode/all_red/source:** All red source.

#### S0012 Police Key
- **/tlc/tc/1/mode/police_key:** Police key active.
- **/tlc/tc/1/mode/police_key/source:** Police key source.

*Note:* Although "All Red" and "Police Key" share the label S0012, they represent distinct modes.