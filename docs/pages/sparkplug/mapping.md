# Mapping

This file lists and status, alarm and commands of the TLC SXL 1.2.1, and define related sparkplug metrics for each.

Metric structure used is <module>/<component>/..

Thes modules are defiend:
- tlc: traffic light control
- traffic: traffic data

These compoent types are defined:
- tc: traffic controller
- sg: signal group
- dl: detector logic
- dh: detector hardware
- in: input
- out: output
- 

## Statuses

### S0001 Signal Group Status
tlc/tc/1/sl/1    string

### S0002 Detector logic status
tlc/tc/1/dl/1    boolean

### S0003 Input status
tlc/tc/1/in/1    string

### S0004 Output status
tlc/tc/1/out/1    string

### S0005 Traffic Light Controller starting
tlc/tc/1/starting   boolean

### S0006 Emergency Route
tlc/tc/1/emergency_route/1   boolean

### S0007 Controller switched on
tlc/tc/1/active           true
tlc/tc/1/active/source    string

### S0008 Manual Control
tlc/tc/1/manual           true
tlc/tc/1/manual/source    string

### S0009 Fixed Time Control
tlc/tc/1/fixed_time           true
tlc/tc/1/fixed_time/source    string

### S0010 Isolated Mode
tlc/tc/1/isolated           true
tlc/tc/1/isolated/source    string

### S0011 Yellow Flash
tlc/tc/1/yellow_flash           true
tlc/tc/1/yellow_flash/source    string

### S0012 All Red
tlc/tc/1/all_red           true
tlc/tc/1/all_red/source    string

### S0012 Police Key
tlc/tc/1/police_key           true
tlc/tc/1/police_key/source    string

### S0014 Time Plan
tlc/tc/1/program           string
tlc/tc/1/program/source    string

### S0014 Traffic Situation
tlc/tc/1/traffic_situation           string
tlc/tc/1/traffic_situation/source    string

### S0016 Number of detector logics
Not needed.

### S00017 Number of signal groups
Not needed.

### S0019 Number of traffic situations
tlc/tc/1/traffic_situation/list      array

### S0020 Control Mode
Not needed.

### S0021 Input Forcing
tlc/tc/1/in/1/force    enum (true, false, release)

### S0022 List of time plans
tlc/tc/1/program/list           array

### S0023 Dynamic Bands
tlc/tc/1/program/1/band/1     integer

### S0024 Offset
tlc/tc/1/program/1/offset     float

### S0025 Time-of-Green / Time-of-Red
tlc/tc/1/sg/1/time_to_green      {min, max, likely, confidence}
tlc/tc/1/sg/1/time_to_red        {min, max, likely, confidence}

### S0026 Week time table
tlc/tc/1/program/schedule/weekdays  {data..}

### S0027 Week time table
tlc/tc/1/program/schedule/hours  {data..}

### S0028
tlc/tc/1/program/1/cycle_time  integer

### S0029 Forced input status
Not needed.

### S0030 Forced output status
tlc/tc/1/out/1/force     enum (true, false, release)

### S0031 Trigger level sensitivity for loop detector
tlc/tc/1/dl/1/treshold      float

### S0032 Coordinated control
tlc/tc/1/coordinated           true
tlc/tc/1/coordinated/source    string

### S0033 Signal Priority Request
tlc/tc/1/sg/1/priority                 {data}
tlc/tc/1/sg/1/priority/queue           array

You send a command and write to /priority.
If successful, the metric is updated. It's also added to the queue which updates the /priority/queue metric. 
Once the status of the request changes /priority/queue is updated.
This means at /priority will always contain the latest request, while /priority/queue contains the current queue.

### S0034 Timeout for dynamic bands
tlc/tc/1/program/1/band/1/timeout     integer

### S0035 Emergency route
Not needed.

### S0091 Operator logged in/out OP-panel
tlc/operator/panel     enum

### S0092 Operator logged in/out OP-panel
tlc/operator/web     enum

### S0095 Version of Traffic Light Controller
sys/version    string

### S0096 Current date and time
sys/clock    datetime

### S0097 Checksum of traffic parameters
tlc/config/checksum    string

### S0098 Configuration of traffic parameters
tlc/config/download           {data}
tlc/config/download/on        boolean

### S0201-S0208 Traffic data
sensor/tc/1/dl/1/events                    {data}     # individual detections    
sensor/tc/1/dl/1/events/throttle           {data}     # control event chunk period
sensor/tc/1/dl/1/aggregated                {data}     # aggregation by time  
sensor/tc/1/dl/1/aggregated/throttle       {data}     # control aggregation period  

events/throttle is used to control how often individual detection events are posted to /events. Events can also
be turned off.
Event can be send indivually as they happen, or can be chunked so that a list of events are send, e.g. each minute.
Chunked data would use the is_historical flag and the timestamp for each events, usign the same metric name for each event.

/aggregated/throttle is used to control how often aggregate values are posted to /aggregated.
Aggregation include e.g. average, minimum and maximum.

If the device is offline and then comes online again, both events and aggregates can be resend using is_historic.


## Alarms

### A0001 Serious hardware error
tlc/hardware/error

### A0002 Less serious hardware error
tlc/hardware/warning

### A0003 Serious configuration error
tlc/config/error

### A0004 Less serious configuration error
tlc/config/warning  

### A0005 Synchronisation error (coordination)
tlc/coordination/error

### A0006 Safety error
tlc/safety/error

### A0007 Communication error
tlc/network/error

### A0008 Deadlock error
tlc/deadlock/error

### A0009 Other error
tlc/error

### A0010 Door open
tlc/door/alarm

### A0101 Pushbutton error
tlc/tc/1/sg/1/pushbutton/error

### A0201 Serious lamp error
tlc/tc/1/sg/1/lamp/error

### A0202 Less serious lamp error
tlc/tc/1/sg/1/lamp/warning

### A0301 Detector error (hardware)
tlc/tc/1/dh/1/error

### A0302 Detector error (logic error)
tlc/tc/1/dl/1/error


## Commands

### M0001 Sets functional position
tlc/tc/1/functional_position     string

### M0002 Sets current time plan
tlc/tc/1/program     integer

### M0003 Sets traffic situation the controller uses
tlc/tc/1/traffic_situation     integer

### M0004 Restarts Traffic Light Controller
tlc/tc/1/starting     boolean

### M0005 Activate emergency route
tlc/tc/1/emergency_route     boolean

### M0006 Activate input
tlc/tc/1/input/1     boolean

### M0007 Activate fixed time control
tlc/tc/1/fixed_time     boolean

### M0008 Sets manual activation of detector logic
tlc/tc/1/dl/1/force     boolean

### M0010 Signal group setStart (Reserved)
Not used.

### M0011 Signal group setStop (Reserved)
Not used.

### M0012 Traffic Light Controller setStart (Reserved)
Not used.

### M0013 Activate a series of inputs
Not needed

### M0014 Set dynamic bands
tlc/tc/1/program/1/bands     {data}

### M0015 Set Offset time
tlc/tc/1/program/1/offset     float

### M0016 Set week time table
tlc/tc/1/program/schedule/weekdays     {data}

### M0017 Set time tables
tlc/tc/1/program/schedule/hours     {data}

### M0018 Set Cycle time
tlc/tc/1/program/1/cycle_time     float

### M0019 Force input
tlc/tc/1/in/1/force     enum (true, false, release)

### M0020 Force output
tlc/tc/1/out/1/force     enum (true, false, release)

### M0021 Set trigger level sensitivity for loop detector
tlc/tc/1/dl/1/treshold     float

### M0022 Request Signal Priority
tlc/tc/1/sg/1/priority                 {data}

### M0023 Set timeout for dynamic bands
tlc/tc/1/program/1/band/1/timeout     integer

### M0103 Set security code
sys/security_code     string

### M0104 Set clock
sys/clock     string


