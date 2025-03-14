## Programs

| Metric                                         | Description                   |
|------------------------------------------------|-------------------------------|
| **/tlc/tc/1/programs**                         | Available programs            |
| **/tlc/tc/1/program**                          | Current program               |
| **/tlc/tc/1/program/1/band**                   | Currently active band         |
| **/tlc/tc/1/program/1/band/extension**         | Band extension                |
| **/tlc/tc/1/program/1/cycle_time**             | Cycle time                    |
| **/tlc/tc/1/program/1/cycle_time/extension**   | Cycle time extension          |
| **/tlc/tc/1/program/1/offset**                 | Offset                        |
| **/tlc/tc/1/program/1/offset/target**          | Target offset                 |
| **/tlc/tc/1/program/1/schedule/weekdays**      | Schedule by day of the week   |
| **/tlc/tc/1/program/1/schedule/hours**         | Schedule by hour of the day   |

### Available Programs
The metric **/tlc/tc/1/programs** provides a list of available programs.

### Program Updates
The metric **/tlc/tc/1/program** indicates the current program.
Change the metric to change the current program.

### Dynamic Bands
The metric **/tlc/tc/1/program/1/band** contains the active band.
The metric **/tlc/tc/1/program/1/band/extension** contains the active band's extension (i.e., duration).
Change this metric to adjust the extension.

### Cycle Time and Offset
The metric **/tlc/tc/1/program/1/cycle_time** reflects the cycle time for the current program. It cannot be modified.
The metric **/tlc/tc/1/program/1/cycle_time/extension** is used to set an extended cycle time. The value provided must be equal to or greater than the base cycle time.

The metric **/tlc/tc/1/program/1/offset** is the current offset.
The metric **/tlc/tc/1/program/1/offset/target** can be used to set a new target offset. The offset will be incrementally shifted over time to reach the target.

### Schedules
Schedules are used to automatically change the program at specific times.

The metric **/tlc/tc/1/program/1/schedule/weekdays** defines the schedule by day of the week.
The metric **/tlc/tc/1/program/1/schedule/hours** defines the schedule by hour of the day.

The hourly schedule overrides the weekly schedule.