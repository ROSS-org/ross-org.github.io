---
layout: post
title:  "GVT-based Sampling"
author: Caitlin Ross
category: instrumentation
---

This collects data immediately after each GVT.
By default, the data is collected on a PE basis, but some metrics can be changed to tracking on a KP or LP basis (depending on the metric).
To turn on instrumentation for the KP/LP granularity, use `--granularity=1`.

When collecting only on a PE basis (i.e., `--granularity=0`), this is the format of the data:

```C
PE_ID, GVT, all_reduce_count, events_processed, events_aborted, events_rolled_back, event_ties, 
total_rollbacks, secondary_rollbacks, fossil_collects, network_sends,
network_recvs, remote_events, efficiency
```

When collecting on a KP/LP basis, this is the format:

```C
PE_ID, GVT, all_reduce_count, events_aborted, event_ties, fossil_collect, efficiency, 
total_rollbacks_KP0, secondary_rollbacks_KP0, ..., total_rollbacks_KPi, secondary_rollbacks_KPi, 
events_processed_LP0, events_rolled_back_LP0, network_sends_LP0, network_recvs_LP0, remote_events_LP0, 
..., events_processed_LPj, events_rolled_back_LPj, network_sends_LPj,network_recvs_LPj,
remote_events_LPj
```

where i,j are the total number of KPs and LPs per PE, respectively.  

You can also choose to sample less often at GVT using the `--num-gvt=n` command, where n is the number of GVT computations to complete between each sampling point.  



### Simulation engine data sampling
To turn on simulation engine sampling, use the option `--engine-stats=n`, where n = 1 for GVT-based sampling, 2 for real time sampling, or 3 to turn on both.  
No model code modifications are needed to collect simulation engine data for any instrumentation mode. 

### Model-level data sampling
To turn on the model-level data sampling, you need to use `--model-stats=n`.
You can turn it on for the GVT-based sampling (`--model-stats=1`), real time sampling (`--model-stats=2`), or both (`--model-stats=3`).
It will sample the model data at the same time as the simulation engine data, depending on the modes you have turned on.
You do not actually need to turn on the simulation engine instrumentation though.
For instance, setting `--model-stats=1` with no other ROSS instrumentation options specified, will sample model data at each GVT, but will not sample the simulation engine data.

