---
layout: post
title:  "Running the Simulator"
author: Caitlin Ross
category: setup
---

## Quick Help ##

Now that you have built ROSS (and possibly learned a bit about building models), change to the `ross-build/models/phold` directory.
You should see several files including a __phold__ binary.  Running the binary with the `--help` flag should give the following output:
```C
odin:phold$ ./phold --help
usage: phold [options] [-- [args]]

PHOLD Model:
  --remote=ts           desired remote event rate (default 0.25)
  --nlp=n               number of LPs per processor (default 8)
  --mean=ts             exponential distribution mean for timestamps (default 1.00)
  --mult=ts             multiplier for event memory allocation (default 1.40)
  --lookahead=ts        lookahead for events (default 1.00)
  --start-events=n      number of initial messages per LP (default 1)
  --stagger=n           Set to 1 to stagger event uniformly across 0 to end time. (default 0)
  --memory=n            additional memory buffers (default 100)
  --run=str             user supplied run name (default undefined)

ROSS MPI Kernel:
  --read-buffer=n       network read buffer size in # of events (default 16)
  --send-buffer=n       network send buffer size in # of events (default 1024)

ROSS Kernel:
  --synch=n             Sychronization Protocol: SEQUENTIAL=1, CONSERVATIVE=2, OPTIMISTIC=3, OPTIMISTIC_DEBUG=4, OPTIMISTIC_REALTIME=5 (default 0)
  --nkp=n               number of kernel processes (KPs) per pe (default 16)
  --end=ts              simulation end timestamp (default 100000.00)
  --batch=n             messages per scheduler block (default 16)
  --extramem=n          Number of extra events allocated per PE. (default 0)
  --buddy-size=n        delta encoding buddy system allocation (2^X) (default 0)
  --lz4-knob=n          LZ4 acceleration factor (higher = faster) (default 17)
  --cons-lookahead=ts   Set g_tw_lookahead (default 0.01)
  --max-opt-lookahead=n Optimistic simulation: maximum lookahead allowed in virtual clock time (default 18446744073709551615)
  --avl-size=n          AVL Tree contains 2^avl-size nodes (default 18)

ROSS MPI GVT:
  --gvt-interval=n      GVT Interval: Iterations through scheduling loop (synch=1,2,3,4), or ms between GVTs (synch=5) (default 16)
  --report-interval=ts  percent of runtime to print GVT (default 0.01)

ROSS Timing:
  --clock-rate=ts       CPU Clock Rate (default 1000000000.00)

ROSS Instrumentation:
  --engine-stats=n      Collect sim engine level stats; 0 don't collect, 1 GVT-sampling, 2 RT sampling, 3 VT sampling, 4 All sampling modes (default 0)
  --model-stats=n       Collect model level stats (requires model-level implementation); 0 don't collect, 1 GVT-sampling, 2 RT sampling, 3 VT sampling, 4 all sampling modes (default 0)
  --num-gvt=n           number of GVT computations between GVT-based sampling points (default 10)
  --rt-interval=n       real time sampling interval in ms (default 1000)
  --vt-interval=ts      Virtual time sampling interval (default 1000000.00)
  --vt-samp-end=ts      End time for virtual time sampling (if different from g_tw_ts_end) (default 0.00)
  --pe-data=n           Turn on/off collection of sim engine data at PE level (default 1)
  --kp-data=n           Turn on/off collection of sim engine data at KP level (default 0)
  --lp-data=n           Turn on/off collection of sim engine data at LP level (default 0)
  --event-trace=n       collect detailed data on all events for specified LPs; 0, no trace, 1 full trace, 2 only events causing rollbacks, 3 only committed events (default 0)
  --stats-prefix=str    prefix for filename(s) for stats output (default )
  --stats-path=str      path to directory to save instrumentation output (default )
  --buffer-size=n       size of buffer in bytes for stats collection (default 8000000)
  --buffer-free=n       percentage of free space left in buffer before writing out at GVT (default 15)
  --disable-output=n    used for perturbation analysis; buffer never dumped to file when 1 (default 0)

Specialized ROSS LPs:
  --sample-count=n      Number of samples to allocate in memory (default 65536)
  --help                show this message
```

## Example ##

There are quite a few options available to us.  Often, only one or two is necessary depending on your platform.
In this case, for example, only one option is required: synch.
The synch option tells ROSS which synchronization protocol to use.
When run with the sequential protocol, I see the following:
```C
odin:phold$ ./phold --synch=1
./phold --sync=1 

Fri May  3 15:23:43 2019

ROSS Version: 7.0.1

tw_net_start: Found world size to be 1 
========================================
PHOLD Model Configuration..............
   Lookahead..............1.000000
   Start-events...........1
   stagger................0
   Mean...................0.000000
   Mult...................1.400000
   Memory.................100
   Remote.................0.250000
========================================


ROSS Core Configuration: 
    Total PEs                                                    1
    Total KPs                                          [Nodes (1) x KPs (16)] 16
    Total LPs                                                    8
    Simulation End Time                                  100000.00
    LP-to-PE Mapping                                        linear


ROSS Event Memory Allocation:
    Model events                                               112
    Network events                                              16
    Total events                                               127

*** START SEQUENTIAL SIMULATION ***

GVT #0: simulation 1% complete, max event queue size 8 (GVT = 1001.0000).
AVL tree size: 0
...(some output removed for brevity)
GVT #0: simulation 99% complete, max event queue size 8 (GVT = 99001.0000).
AVL tree size: 0
*** END SIMULATION ***


    : Running Time = 0.3076 seconds

TW Library Statistics:
    Total Events Processed                                  799992
    Events Aborted (part of RBs)                                 0
    Events Rolled Back                                           0
    Event Ties Detected in PE Queues                        699993
    Efficiency                                              100.00 %
    Total Remote (shared mem) Events Processed                   0
    Percent Remote Events                                     0.00 %
    Total Remote (network) Events Processed                      0
    Percent Remote Events                                     0.00 %

    Total Roll Backs                                             0
    Primary Roll Backs                                           0
    Secondary Roll Backs                                         0
    Fossil Collect Attempts                                      0
    Total GVT Computations                                       0

    Net Events Processed                                    799992
    Event Rate (events/sec)                              2600585.1
    Total Events Scheduled Past End Time                         8

TW Memory Statistics:
    Events Allocated                                           128
    Memory Allocated                                            77
    Memory Wasted                                              434

TW Data Structure sizes in bytes (sizeof):
    PE struct                                                  624
    KP struct                                                  144
    LP struct                                                  136
    LP Model struct                                              8
    LP RNGs                                                     80
    Total LP                                                   224
    Event struct                                               152
    Event struct with Model                                    160

TW Clock Cycle Statistics (MAX values in secs at 1.0000 GHz):
    Initialization                                          0.0144
    Priority Queue (enq/deq)                                0.1135
    AVL Tree (insert/delete)                                0.0000
    LZ4 (de)compression                                     0.0000
    Buddy system                                            0.0000
    Event Processing                                        0.0000
    Event Cancel                                            0.0000
    Event Abort                                             0.0000

    GVT                                                     0.0000
    Fossil Collect                                          0.0000
    Primary Rollbacks                                       0.0000
    Network Read                                            0.0000
    Other Network                                           0.0000
    Instrumentation (computation)                           0.0000
    Instrumentation (write)                                 0.0000
    Total Time (Note: Using Running Time above for Speedup)      0.7998

TW GVT Statistics: MPI AllReduce
    GVT Interval                                                16
    GVT Real Time Interval (cycles)                    0
    GVT Real Time Interval (sec)                        0.00000000
    Batch Size                                                  16

    Forced GVT                                                   0
    Total GVT Computations                                       0
    Total All Reduce Calls                                       0
    Average Reduction / GVT                                   -nan
```

## Options ##

### ROSS Kernel Options ###

#### `--synch={1,2,3,4,5}` ####
The `synch` option can take the following values:
* 1 (sequential): Instructs ROSS to use a sequential engine, i.e. the simulation will consist of only one processor (PE).  
* 2 (conservative): Instructs ROSS to use a conservative engine, i.e. all LPs will have the same notion of virtual time.  
* 3 (optimistic): Instructs ROSS to use an optimistic engine, i.e. LPs can have varying notions of virtual time, though rollbacks will be possible and a reverse event handler will be required in this case.
* 4 (optimistic debug): Executes ROSS in a serial simulation until it runs out of memory. Then it rolls every message back. This can be used to test rollback functions.
* 5 (optimistic real time): Similar to optimistic, but GVT is triggered based on the amount of time elapsed rather than number of events processed.
See [this post](https://ross-org.github.io/feature/synch5.html) for details.

#### `--nkp=n` ####
* `nkp` specifies the number of Kernel Processes (KPs) we have per Processing Element (PE).  LPs are mapped onto KPs which aggregate the events processed by those LPs.

#### `--end=ts` ####
* `end` specifies the final timestamp to process.  Discrete event simulations (typically) schedule a new event from within the event handler.  Therefore, in the general case, it will never stop unless the simulator notices that it has simulated past the point of interest.

#### `--batch=n` ####
`batch` is the number of events to be processed in an iteration of the main scheduling loop. This can control how often you poll the network for events (i.e., the smaller the number, the more often you poll the network).

#### `--extramem=n` ####
* `extramem` specifies the amount of additional memory to allocate.  This helps during simulations as we never dynamically allocate memory at runtime; we just perform a GVT and reclaim event memory.  If you see a high number in the "Forced GVT" output above, try increasing this value.  A good starting value for _n_ would be 2 * batch * GVT.

