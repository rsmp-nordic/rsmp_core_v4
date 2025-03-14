## Detector Logics

Metrics used for detector logics, including status, treshold and forcing.

| Metric                           | Description              |
|----------------------------------|--------------------------|
| **tlc/tc/1/dl/1**                | Detector status          |
| **tlc/tc/1/dl/1/threshold**      | Detector threshold       |
| **tlc/tc/1/dl/force**            | Force logic evaluation   |
| **tlc/tc/1/dl/1/error**          | Detector error           |

### Detector Logic Status
The metric **tlc/tc/1/dl/1** represents the status of the active detector logic.  
A "true" value means the logic is engaged; "false" means it is inactive.

### Detector Trigger Threshold
The metric **tlc/tc/1/dl/1/threshold** sets the trigger level for the detector logic.  
This value determines when the logic activates based on input conditions.

### Forcing Detector Logic Evaluation
The metric **tlc/tc/1/dl/force** forces the detector logic to run immediately.  
This is useful for testing or resetting without waiting for normal triggers.

### Detector Alarms
The metric **tlc/tc/1/dl/1/error** indicates if an error alarm is raised by the detector logic.  
An active alarm signals that the detector's conditions have triggered an error response.