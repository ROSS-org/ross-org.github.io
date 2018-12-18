---
layout: post
title: Optimistic Parameters
author: Caitlin Ross
category: feature
---

There are several different parameters that are available for tuning optimistic parallel mode in ROSS.
Optimistic mode can be performed in two ways, the difference between the two ways is how the frequency of GVT computation is determined.
The traditional way, `--sync=3`, uses the number of events to determine how frequently to perform GVT.
This way means GVT will be performed after every `batch` x `GVT-interval` number of events (these parameters are explained in more detail below).
The other method is similar, but GVT is performed after some amount of real time has passed (see [Optimistic Realtime Scheduler](http://ross-org.github.io/feature/synch5.html) for more details).

### GVT-interval
The `--gvt-interval` parameter determines either the number of iterations through the main scheduling loop (`--sync=3`), or the amount of real time in milliseconds between GVT computations (`--sync=5`).
In general, each iteration of the main scheduling loop performs the following actions:

- check network for new events received
- put events into priority queue
- check for events in cancelation queue and handle appropriately
- perform GVT if it's time to do so
- batch event processing

Some good starting points for `--gvt-interval` are:
- for `--sync=3`: 128 or 256
- for `--synch=5`: 32 or 64

If you experience a lot of rollbacks with your model, it may be a good idea to reduce the GVT-interval.

### Batch
The `--batch` parameter determines the number of events processed during the batch event processing listed above in the main scheduling loop.
The main thing here to note, is that it is the number of forward events to process between each network poll for new events.
If you have poor rollback behavior, you may consider keeping this really low, at values such as 1 or 2.
This way you will be checking the network often for new events that have come in and hopefully this can help reduce the events that cause rollbacks.
If you have pretty good optimistic performance in terms of rollbacks, you may consider increasing `--batch`.

### Number of Kernel Processes
Kernel Proceses (or KPs) were added to ROSS to help reduce fossil collection overhead.
On a given PE, each KP is responsible for a subset of that PE's LPs.
The processed event list (which is used to support the rollback mechanism) is stored on a KP basis instead of LP basis.
This helps to reduce the fossil collection overhead, but can potentially increase rollbacks, because when one LP on a KP needs to rollback, all other LPs on that same KP must rollback.
Essentially fewer KPs means an increased chance of false rollbacks.
If your model has poor rollback behavior, you may try increasing the number of KPs via `--nkp` (Note: This value is the number of KPs per PE, not total KPs for the simulation).
It will probably increase the time spent in fossil collection, but for a model with poor rollback behavior, it may not be much compared to the overhead of excessive rollbacks.


### Max Optimistic Lookahead
Max optimistic lookahead puts a window on the events that can be processed.
When this is set, only events with timestamps between GVT and `--max-opt-lookahead` will be processed.
This value is set globally for all PEs and the window is checked on a PE basis.
When a PE determines it has no events in this window to process, it will be forced into GVT processing and wait at the first `MPI_Allreduce` for everyone else to catch up.
Setting this value may be useful when you have a load imbalance in your simulation, as it will keep PEs with lighter loads from getting too far ahead in virtual time and needing to be rolled back from messages from PEs with heavy loads.

The value to set `--max-opt-lookahead` is model dependent.
Currently, we are working on a way to set this automatically in the simulation, but this is not available yet.
