---
layout: post
title:  "Data Sample Format"
author: Caitlin Ross
category: instrumentation
---

Since the instrumentation data is output in binary, this page describes the format of the data.
You can also see the [ROSS Binary Reader repo](https://github.com/caitlinross/ross-binary-reader) for examples of reading the data.
Each sample collected (regardless of which instrumentation mode is used) is broken into two parts, the metadata and the sample data.

### Metadata
The metadata for all samples (except event data), whether it is model data or simulation engine data, starts as follows:

```C
int sample_type        // 0 = PE, 1 = KP, 2 = LP, 3 = Model sample
int sample_size        // size of the data in this sample
double virtual_time    // virtual time at sampling
double real_time       // real time at sampling
```

For model level data, there is another set of metadata:
```C
unsigned int pe_id
unsigned int kp_id
unsigned int lp_id
float gvt
int stats_type
int model_size
```

This keeps track of PE, KP, and LP ids so models don't have to (because some models may prefer to keep track of some kind of local LP ids instead).
The GVT value is so the last GVT can be tracked.
The `sample_size` recorded in the first part of the metadata refers to the size of this model metadata, and the `model_size` here refers to the actual size of the model data saved in each sample.

##### Event Metadata

The event metadata is structured as follows:

```C
unsigned int source_lp
unsigned int destination_lp
float virtual_send_time
float virtual_recv_time
float real_time // when this event data was collected
unsigned int model_data_size
```

The model data immediately follows and is in the format determined by the model.

### Sample Data
Model data is of course set by the model, so you will have to look at that specific model to determine how that sample is formatted.
The next sections describe the format for simulation engine data at the PE, KP and LP levels.

##### PE
```C
unsigned int pe_id
unsigned int events_processed
unsigned int events_aborted
unsigned int events_rolled_back
unsigned int total_rollbacks
unsigned int secondary_rollbacks
unsigned int fossil_collect_attempts
unsigned int priority_queue_size
unsigned int network_sends
unsigned int network_receives
unsigned int num_GVTs
unsigned int pe_event_ties
unsigned int all_reduce_count

float efficiency
float network_read_time
float network_other_time
float GVT_time
float fossil_collect_time
float events_aborted_time
float events_processed_time
float priority_queue_time
float rollback_time
float cancel_q_time
float avl_tree_time
float buddy_time
float lz4_time
```
All of the `unsigned int` metrics are counters.
All of the `float` metrics that end with the word "time" track how long that PE spent doing that type of processing during the sampling interval.

##### KP
```C
unsigned int pe_id
unsigned int kp_id
unsigned int events_processed
unsigned int events_aborted
unsigned int events_rolled_back
unsigned int total_rollbacks
unsigned int secondary_rollbacks
unsigned int network_sends
unsigned int network_receives

float time_ahead_gvt
float efficiency
```

This has most of the same data as the PE level, except some metrics are only relevant at the PE level, so they aren't included here.
There is a new metric here, which is `time_ahead_gvt`. 
This is the difference between a KP's local clock and the current GVT at the time of sampling.

##### LP

The LP samples are almost exactly like the KP, except they do not track the difference between local clocks and GVT (since ROSS keeps the local clocks on the KP level and not the LP level).

```C
unsigned int pe_id
unsigned int kp_id
unsigned int lp_id
unsigned int events_processed
unsigned int events_aborted
unsigned int events_rolled_back
unsigned int network_sends
unsigned int network_receives

float efficiency
```

