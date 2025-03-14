## Sensor Data

| Metric                                         | Description                          |
|------------------------------------------------|--------------------------------------|
| **sensor/tc/1/dl/1/events**                    | Individual detection events          |
| **sensor/tc/1/dl/1/events/throttle**           | Event chunk period control           |
| **sensor/tc/1/dl/1/aggregated**                | Time-based aggregated detections     |
| **sensor/tc/1/dl/1/aggregated/throttle**       | Aggregation period control           |

### Detection Events
The metric **sensor/tc/1/dl/1/events** captures each individual detection event as data.  
These events represent the raw detections reported by the sensor.

The metric **sensor/tc/1/dl/1/events/throttle** indicates the period over which individual events are chunked together.  
Adjust this metric to manage the frequency of event reporting. The individual events will be send as single data 
data message with multiple version of the same metric, with the is_historic flag set and corresponding timestamps.

### Aggregation
The metric **sensor/tc/1/dl/1/aggregated** provides aggregated detection data over a specified time interval.  

The metric **sensor/tc/1/dl/1/aggregated/throttle** controls the aggregation period.  
Modifying this value adjusts how often the sensor produces aggregated output.
The data mesage will contain a single metric with the timestamp set to when the aggregation was done.
