---
layout: post
title:  "Instrumentation Overview"
author: Caitlin Ross
category: instrumentation
---

There are several different modes of instrumentation have been added to ROSS that can be used to collect data on the simulation engine and/or the model being simulated: 

* [GVT-based sampling](gvt-sampling.html)
* [real time sampling](real-time-sampling.html)
* [virtual time sampling](virtual-time-sampling.html)
* [event tracing](event-tracing.html)

All instrumentation modes can be used independently or together.
The command line options for the instrumentation are shown under the title "ROSS Instrumentation" when you run `--help` with a ROSS/CODES model.
For all instrumentation types, you can use `--stats-filename` option to set a prefix for the output files, otherwise the default is "ross-stats".
By default, all of the output files are stored in a directory named `stats-output` that is created in the running directory.
With the option `--stats-path` you can set the path to save the instrumentation files.
In either case, if the directory already exists, ROSS will append a suffix of numbers to the end, so data is not overwritten.

To sample simulation engine data in any instrumentation mode, no code changes are needed.
Collecting model-level data does require adding in some code to the model(s), but we've attempted to keep this as simple as possible.
The details on how to set this up are on this page.

### Simulation engine data sampling
To turn on simulation engine sampling, use the option `--engine-stats=n`, where n = 1 for GVT-based sampling, 2 for real time sampling, or 3 to turn on both.
As stated previously, no model code modifications are needed to collect simulation engine data for any instrumentation mode.

NOTE: The `--engine-stats` option does not currently apply to the [Virtual Time Sampling](virtual-time-sampling.html). 
See that page for more details.

### Model-level data sampling
To turn on the model-level data sampling, you need to use `--model-stats=n`.
You can turn it on for the GVT-based sampling (`--model-stats=1`), real time sampling (`--model-stats=2`), or both (`--model-stats=3`).
For instance, setting `--model-stats=1` with no other ROSS instrumentation options specified, will sample model data at each GVT, but will not sample the simulation engine data.

NOTE: The `--model-stats` option does not currently apply to the [Virtual Time Sampling](virtual-time-sampling.html). 
See that page for more details.

#### Model Setup
Registering the function pointers necessary for the callback functions required for the model-level sampling is implemented similarly to how the LP type function pointers are implemented in ROSS core.
(As a side note, it is a different struct, so that non-instrumented ROSS can still be run without requiring any additions to models).

##### Model types struct 
```C
typedef struct st_model_types st_model_types;
struct st_model_types {
    rbev_trace_f rbev_trace; /* function pointer to collect data about events causing rollbacks */
    size_t rbev_sz;          /* size of data collected from model about events causing rollbacks */
    ev_trace_f ev_trace;     /* function pointer to collect data about all events for given LP */
    size_t ev_sz;            /* size of data collected from model for each event */
    model_stat_f model_stat_fn; /* function pointer to collect model-level data */
    size_t mstat_sz;         /* size of data collected from model at sampling points */
    sample_event_f sample_event_fn; /* function pointer for virtual time sampling of model data */
    sample_revent_f sample_revent_fn; /* RC function pointer for virtual time sampling of model data */
    size_t sample_struct_sz; /* size of data collected using virtual time sampling */
};
```
This is the struct where we provide the function pointers to ROSS (details on the functions are below).
`rbev_trace` is the pointer for the event collection for only events causing rollbacks, while `ev_trace` is for the full event collection.
`rbev_sz` and `ev_sz` are the sizes of the data that need to be pushed to the buffer for the rollback causing events, or for all events, respectively.
`model_stat_fn` is the pointer for collecting model-level data with the GVT-based and real time sampling modes.
Similarly, `mstat_sz` is the size of the data that will be saved at each sampling point for that LP.
`sample_struct_sz` is the size of the model data that will be saved at each virtual time sampling point.

##### Function pointers:
```C
typedef void (*rbev_trace_f) (void *msg, tw_lp *lp, char *buffer, int *collect_flag); // event tracing
typedef void (*ev_trace_f) (void *msg, tw_lp *lp, char *buffer, int *collect_flag);   // event tracing
typedef void (*model_stat_f) (void *sv, tw_lp *lp, char *buffer);                     // real time or GVT-based sampling
typedef void (*sample_event_f) (void *state, tw_bf *b, tw_lp *lp, void *sample);      // virtual time sampling
typedef void (*sample_revent_f) (void *state, tw_bf *b, tw_lp *lp, void *sample);     // virtual time sampling
```

The first two functions are for the event tracing functionality.
`msg` is the message being passed, `lp` is the LP pointer.
`buffer` is the pointer to where the data needs to be copied for ROSS to manage.
ROSS will allocate the correct amount of space based on the rbev_sz or ev_sz provided in the `st_model_types` struct.
`collect_flag` is a pointer to a ROSS flag.  By default `*collect_flag == 1`, meaning the event will be collected.
This flag was added to let you control whether specific events for an LP are collected or not.
Change the value to 0 for any events you do not want to show in the trace.
This means even the ROSS level data will not be collected for that event (see [Event Tracing](event-tracing.html) for details).
You can use this feature to turn off event tracing for certain events, reducing the amount of data you will be storing.

The third function is for model-level data to be sampled at the same time as the simulation engine data (at GVT or based on real time). 
`sv` is the pointer to the LP state. 

The last two functions are for virtual time sampling and provide pointers to the LP state, the bit field, LP, and the data to be collected at this sampling point.
ROSS allocates the space for `sample` based on the value of `sample_struct_sz` provided in `st_model_types`.
If you don't change LP state any in the forward event, you probably do not need to implement the reverse handler, but see [Virtual Time Sampling](virtual-time-sampling.html) for more details.

##### Register Function Pointers
There are two ways of registering your pointer functions with ROSS depending on the way you register your LP types with ROSS:  

###### Method 1
If you call the `tw_lp_settype()` for LP type registration with ROSS, you should follow this method.

To register the function pointers with ROSS, call `st_model_settype(tw_lpid i, st_trace_type *trace_types)` right after you call the `tw_lp_settype()` function when initializing your LPs.
You can also choose to turn event tracing on for only certain LPs.
To do this, you only need to call `st_model_settype()` with the appropriate arguments for the LPs you want event tracing turned on.

For example, here is how it is done in PHOLD in `main()`:

```C
for (i = 0; i < g_tw_nlp; i++)
{
    tw_lp_settype(i, &mylps[0]);
    st_model_settype(i, &model_types[0]);
}
```

`mylps` is the `tw_lptype` struct containing the usual ROSS function pointers for initialization, event handlers, etc.
`model_types` is the `st_model_types` struct described above.  

If your model is a part of CODES, the CODES mapping will handle this for you.
See the [CODES repo](https://xgitlab.cels.anl.gov/codes/codes) and documentation for more details.  

###### Method 2
If you call the `tw_lp_setup_types()` function for LP registration, follow this method.
This requires that you set `g_tw_lp_types` and `g_tw_lp_typemap`.

Similarly for registering the model sampling functions, you'll need to set `st_model_types *g_st_model_types`.
This needs to be set **before** the call to `tw_lp_setup_types()`.
If you have multiple LP types that you are registering, the array of `st_model_types` should be in the same order as the `tw_lptypes` array that you set `g_tw_lp_types` to point to.  

That's it.  The call to `tw_lp_setup_types()` will make the call to `st_model_setup_types()` if you have anything requiring model-level sampling turned on.  



### Output formatting
All collected data is pushed to a buffer as it is collected, in order to reduce the amount of I/O accesses.
Currently the buffer is per PE.
If multiple instrumentation modes are used, each has its own buffer.
The default buffer size is 8 MB but this can be changed using `--buffer-size=n`, where n is the size of the buffer in bytes.
If you have enough memory on the nodes you run your simulations on, you should probably increase the buffer size to write out less often.

After GVT, the buffer's free space is checked.
By default, if there is less than 15% free space in the buffer, then it is dumped to file in a binary format.
This can be changed using `--buffer-free=n`, where n is the percentage of free space it checks for before writing out.  

The output is in binary and right now it outputs one file per simulation per instrumentation type
(e.g., if you run both GVT-based and real time instrumentation, you get a file with the GVT data and a 2nd file for the real time sampling).
ROSS will create a directory called stats-output that these files will be placed in.

There is a basic binary reader for all of the instrumentation modes being developed in the 
[ROSS Binary Reader repo](https://github.com/caitlinross/ross-binary-reader).
For the time being, ROSS will output a README file in the stats-output directory with the given filename prefix.
The file contains some general information about values of input parameters, but also has data that the reader can use to correctly read the instrumentation data.

### Other notes
There are a couple of other options that show up in the ROSS stats options.
One is `--disable-output=1`.
This is typically only used to examine the perturbation of the instrumentation on the simulation.
It means that data will be pushed to the buffer, but the buffer will never be dumped, so it will just keep overwriting data.
This is so we can measure the effects of the instrumentation layer itself without the I/O. 
Typically you'll want to leave this turned off.
At some point in the future, this will probably be converted into allowing data to be streamed to an in situ analysis system.  


