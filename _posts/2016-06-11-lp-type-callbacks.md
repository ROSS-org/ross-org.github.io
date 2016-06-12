---
layout: post
title:  LP Type Callbacks
author: Elsa Gonsiorowski
category: model-dev
---

The functionality of an LP is realized through a number of callback functions and LP type can have its own set of callbacks.
As the model developer, you simply need to program a single LP's task for each function.
The ROSS simulation engine takes care of the rest.

The callback functions are:

1. Initialization function
2. Pre-run function (optional)
2. Forward event handler
2. Reverse event handler (required for optimistic simulation)
2. Commit event function (optional)
2. Finalize function

## Callbacks

The initialize, pre-run, and finalize functions are called outside of simulation.
These functions are called once for each LP.
The function arguments are the model defined LP state struct and the ROSS internal LP state representation.

The forward, reverse, and commit event functions are called during simulation.
They are associated with both an LP and an event.
In addition to the model and ROSS LP state variables, the model defined message and a bit field variable are passed as arguments.

### Initialization

```
    void init_f(model_state * s, tw_lp * lp);
```

This callback is given an freshly allocated LP state struct along with the ROSS LP struct (type `tw_lp`).
During this function, all the variables in the LP state should be initialized.
Through the LP state struct, model developers have access to `lp->gid`, which is the globally unique ID for this LP.

Model developers may also want to create some initial events to kick-off the simulation.
**Caveat:** During initialization, LPs can only send events to themselves.

### Pre-Run

```
    void pre_run_f(model_state * s, tw_lp * lp);
```

The self-scheduled-only event caveat of the initialization function lead to the creation of the pre-run callback.
Here, LPs are able to schedule events for any other LP in the system.

### Forward Event Handler

```
    void event_f(model_state * s, tw_bf * bf, msg_type * in_msg, tw_lp * lp);
```

The forward event handler is triggered by an event arriving at the LP.
There is one event handler per LP type, so it must by able to process any event type that arrives.
Forward event handlers typically schedule more events and make calls to the random number generator.

### Reverse Event Handler

```
    void reverse_f(model_state * s, tw_bf * bf, msg_type * in_msg, tw_lp * lp);
```

The reverse event handler should undo any state changes made by the forward event handler.
It is best practice to make sure that both the forward and reverse event handlers have the same structure and control flow.

The model developer is responsible for reversing the random number generator (see [this post](http://carothersc.github.io/ROSS/feature/random-numbers.html)).
ROSS takes cares of undoing any events that were sent by the forward event handler.

### Commit Event Handler

```
    void commit_f(model_state * s, tw_bf * bf, msg_type * in_msg, tw_lp * lp);
```

The commit event handler is called for an event on LP at commit time.
For sequential and conservative simulations, this is immediately following the forward event handler.
For optimistic simulations, the commit function is called once GVT has passed the event receive time, when there it is guaranteed to not be rolled back.

The commit function can be used for print statements or update model state.

### Finalize

```
    void final_f(model_state * s, tw_lp * lp);
```

The LP finalize is called for each LP after the simulation has ended.
It is typically used to print out final LP statistics.

## LP Type

The callback functions are collected into a `tw_lptype` array (note the order of functions).
This array includes a basic mapping function and the size of the model state struct (*mapping details will be covered in a different post*).

The pre-run and commit functions are optional (and should be replaced with `NULL` if not implemented by the model developer).
All other function callbacks are required and must be implemented by the model.

An example of the `tw_lptype` struct:

```
    tw_lptype model_lps[] = {
        {   (init_f) model_init,
            (pre_run_f) NULL,
            (event_f) model_event,
            (revent_f) model_reverse,
            (commit_f) model_commit,
            (final_f) model_final,
            (map_f) model_map,
            sizeof(model_state)  },
        { 0 },
    };
```

This struct should be assigned to the global variable `g_tw_lp_types` in the main function before `tw_lp_setup_types()` is called.
