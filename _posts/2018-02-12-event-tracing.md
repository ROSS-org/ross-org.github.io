---
layout: post
title:  "Event Tracing"
author: Caitlin Ross
category: instrumentation
---

For event tracing, ROSS can directly access the source and destination LP IDs for each event, as well as the sent and received virtual timestamps.
It will also record the real time that the event is computed at.

Because event types are determined by the model developer, ROSS cannot directly access this.
The user can create a callback function that ROSS will use to collect the event type, and ROSS will then handle storing the data in the buffer and the I/O.
The benefit to this is that the user can choose to collect other data about the event, and ROSS will handle this as well.
It's important to remember that this data collection will result in a lot of output, since it's happening per event.

There are two ways to collect the event trace.
One is to collect data only about events that are causing rollbacks.
When an event that should have been processed in the past is received, data about this event is collected.
The other method of event tracing is to collect data for all events.
To turn on the event tracing with either `--event-trace=1` for full event trace or `--event-trace=2` for tracing only events that cause rollbacks. 
For either method, individual events can be set to not be collected by setting `collect_flag=0` in the `ev_trace_f` and/or `rbev_trace_f` functions (see [Instrumentation Overview](instrumentation.html) for details).
`collect_flag` is set to 1 by default, so it is only necessary to change if you don't want to collect an event in some scenario.
Setting the flag to 0 means that no data at all will be collected for this event, including the data directly accessed by ROSS described above.



