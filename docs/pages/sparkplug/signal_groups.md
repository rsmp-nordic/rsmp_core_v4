# Signal Group Metrics

Metrics related to a signal groups, including current and predicted states and signal priority.

| Metric                              | Description                          |
|-------------------------------------|--------------------------------------|
| **/tlc/tc/1/sg/1**									| Current signal group status          |
| **/tlc/tc/1/sg/1/prediction**       | State prediction 										 |
| **/tlc/tc/1/sg/1/priority**         | Priority data for the signal group   |
| **/tlc/tc/1/sg/1/priority/queue**   | Signal group priority queue (array)  |

### Signal Group Status
The metric **/tlc/tc/1/sg/1** provides the stage of the signal group.

### Prediction
Predictions about future state changes is provided in **/tlc/tc/1/sg/1/prediction** including minimum, maximum and most like times, as well as a confidence meassure.
This data can be used green light optimized speed advice (GLOSA) applications.

### Signal Priority
The metric **/tlc/tc/1/sg/1/priority** indicates the latest singal priority request.
The metric **/tlc/tc/1/sg/1/priority/queue** contains a list of active priority requests.

To request priority, send a command to modift write to **/tlc/tc/1/sg/1/priority**
If successful, the metric is updated. It's also added to the queue which updates **/tlc/tc/1/sg/1/priority/queue**. 
As the status of the request changes **/tlc/tc/1/sg/1/priority/queue** is updated.
