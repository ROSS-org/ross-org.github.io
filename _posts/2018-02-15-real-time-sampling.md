---
layout: post
title:  "Real Time Sampling"
author: Caitlin Ross
category: instrumentation
---

This collects data at real time intervals specified by the user.
The runtime option `--rt-interval=n` sets the sampling interval, where n is the number of milliseconds between sampling points.
The default is set to 1000 ms.
This collects all of the same data as the GVT-based instrumentation, as well as some other metrics, which is the difference in GVT and local virtual time for each KP as well as the cycle counters for the PEs (e.g., how much time is spent in event processing, roll back processing, GVT, etc).
The difference in GVT and KP virtual time is always recorded per KP, regardless of how `--granularity` is set.
For the other metrics, the granularity can be switched as described in the section on the GVT-based instrumentation.

When collecting on a PE basis, this is the data format:

```C
PE_ID, current_real_time, current_GVT, time_ahead_GVT_KP0, ..., time_ahead_GVT_KPi,
network_read_CC, gvt_CC, fossil_collect_CC, event_abort_CC, event_processing_CC,
priority_queue_CC, rollbacks_CC, cancelq_CC, avl_CC, buddy_CC, lz4_CC,
events_aborted, pq_size, remote_events, network_sends, network_recvs,
event_ties, fossil_collects, num_GVT, events_processed, events_rolled_back,
total_rollbacks, secondary_rollbacks 
```

For the KP/LP granularity:

```C
PE_ID, current_real_time, current_GVT, time_ahead_GVT_KP0, ..., time_ahead_GVT_KPi,
network_read_CC, gvt_CC, fossil_collect_CC, event_abort_CC, event_processing_CC,
priority_queue_CC, rollbacks_CC, cancelq_CC, avl_CC, buddy_CC, lz4_CC,
events_aborted, pq_size, event_ties, fossil_collects, num_GVT,
total_rollbacks_KP0, secondary_rollbacks_KP0, ..., total_rollbacks_KPi, secondary_rollbacks_KPi,
events_processed_LP0, events_rolled_back_LP0, network_sends_LP0, network_recvs_LP0, remote_events_LP0,
..., events_processed_LPj, events_rolled_back_LPj, network_sends_LPj,network_recvs_LPj,
remote_events_LPj
```

### Simulation engine data sampling
To turn on simulation engine sampling, use the option `--engine-stats=n`, where n = 1 for GVT-based sampling, 2 for real time sampling, or 3 to turn on both.
No model code modifications are needed to collect simulation engine data for any instrumentation mode. 

### Model-level data sampling
To turn on the model-level data sampling, you need to use `--model-stats=n`.
You can turn it on for the GVT-based sampling (`--model-stats=1`), real time sampling (`--model-stats=2`), or both (`--model-stats=3`). 
It will sample the model data at the same time as the simulation engine data, depending on the modes you have turned on.
You do not actually need to turn on the simulation engine instrumentation though.
For instance, setting `--model-stats=1` with no other ROSS instrumentation options specified, will sample model data at each GVT, but will not sample the simulation engine data.


