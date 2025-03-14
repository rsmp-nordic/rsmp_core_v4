## Interface Inputs and Outputs

| Metric                          | Description                |
|---------------------------------|----------------------------|
| `tlc/tc/1/in/1`                 | Current input state        |
| `tlc/tc/1/in/1/force`           | Input forcing              |
| `tlc/tc/1/out/1`                | Current output state       |
| `tlc/tc/1/out/1/force`          | Output forcing             |

### Input Overview
The metric `tlc/tc/1/in/1` reflects the current state of the interface input.  
You can write to this metric to set the input; however, doing so does not lock the state, which means other processes might update it.

To lock the input state, set the metric `tlc/tc/1/in/1/force` to true.
When forced, the input remains at the specified state until the force is released.

### Output Overview
The metric `tlc/tc/1/out/1` reflects the current state of the interface output.  
You can write to this metric to set the output; however, writing to it without forcing does not lock its state, allowing updates from other processes.

To lock the output state, set the metric `tlc/tc/1/out/1/force` to true.  
When forced, the output remains in the specified state until the forcing is explicitly released.