---
layout: post
title:  Debugging Tips & Tricks
category: model-dev
---

Debugging a PDES model can be a very tricky venture.
Fortunately, there seem to be a small subset of common bugs that are found in many different models.
This post discusses the basics of debugging a parallel program and some of the bugs to watch out for.
Learn from mistakes of the past!

*If you have run across any bugs which you think should be included here,* **Please** *write up a brief description of your bug and include it in a [pull request](https://github.com/carothersc/ROSS/pulls) or [GitHub issue](https://github.com/carothersc/ROSS/issues).*

## When is a Model Correct?

Any valid PDES model should have one simple property: <mark>determinism</mark>.
That is, performing the same simulation using different synchronization algorithms or parallel configurations should yield the same <mark>net event count</mark>.

Typically, the model development process is:

1. Create a model which works using sequential execution.
2. Run the model in parallel using conservative simulation and verify that the results match with serial execution.
3. Write the reverse event handlers for optimistic simulation.
4. Debug the optimistic simulation until the result match the conservative and sequential experiments.

## Debugging in Parallel

Parallel debugging is a non-trivial task.
Coupled with optimistic PDES, the very trickiest bugs may disappear when the developer makes even a small change code.
Hopefully, most bugs can be produced in a small 2-way or 4-way parallel experiment.

### Print statements

Parallel programs can be large and complex.
Despite this fact, print statements are the quickest and easiest way to gain some understanding of what is happening during execution.
Often, when debugging event handlers, you should print the LP's GID and the event type it is currently processing.

To create cleaner output files (without parallel output messages crossing each other), use one file per MPI rank.
This can be easily done with the following code snippet:

```C
    // global file pointer
    FILE * node_out_file;

    // create and open one file per MPI rank
    // done in main
        char *dpath = dirname(argv[0]);
        char debugfilename[100];
        sprintf(debugfilename, "%s/node_%d_output_file.txt", dpath, g_tw_mynode);
        node_out_file = fopen(debugfilename,"w");

    //use the file during LP event handling
        fprintf(node_out_file, "FWDE: #%u %d %2.3f %u\n", lp->gid, in_msg->type, tw_now(lp), in_msg->id);
        fflush(node_out_file);

    //use the file during the reverse event
        fprintf(node_out_file, "REVE: #%u %d %2.3f %u\n", lp->gid, in_msg->type, tw_now(lp), in_msg->id);
        fflush(node_out_file);

    // you should probably close the files when you are done (in main)
        node_out_file.close();
```

A simple python script can be created to analyze the debug output.
Matching the output from an optimistic experiment to the same serial execution may help pin point the issue.

### GDB

[X Windows](https://en.wikipedia.org/wiki/X_Window_System) is a UNIX GUI system.
It can be spawned from the command line using an `mpirun` call.
This is how GDB is instantiated in parallel:

```
    $ mpirun -np 2 xterm -e gdb ./model
```

**Note for Mac Systems**: even on a Mac system you will probably want to download and install GDB (rather than use LLDB).
This can be done through [Homebrew](http://brew.sh).
You will also need to codesign GDB to allow it to run, see [this page](https://sourceware.org/gdb/wiki/BuildingOnDarwin) for more details.

### XCode and other IDEs

#### Xcode

* Set your ARCH environment variable: `export ARCH=x86_64` for example
* Configure your project (using cmake) to generate Xcode projects with the proper generator:
`cmake -G Xcode <path>`
* Once it has been created, you can open it like so: `open <proj_name>.xcodeproj`
* In the Project Navigator, in the ROSS target, add to `Header Search Paths` the appropriate folder containing your MPI headers.  You should now be able to build the ROSS library
* In your model, add to `Header Search Paths` the same folder as above. Additionally, add the appropriate folder to `Library Search Paths` containing the path to your MPI libraries. Also, add `-lmpi` to your `Other Linker Flags` settings for your build type (e.g., Debug, Release, etc.)  You should now be able to build your model

The Xcode-included debugger is serial only.
In order to debug a parallel program we must start the program from the Terminal with our standard `mpirun -np <num> ...` incantation but we must "catch it" somehow.
One simple approach is to put a bogus `while` loop at the top of your `main()` function:

```
    int i = 1;
    while (i)
       continue;
```

You can then use the Xcode debugger to attach to the MPI processes (either by `Debug->Attach to Process by PID or Name` or `Debug->Attach to Process`) and change the value of `i` to `0` on each proess to continue running your code.

## Bug 1: Conditionals using Dynamic State Variables

It is common to use the current LP state in a branching statement and then later change update the same state variable.
During a reverse event, the branching statement must be changed to reflect the variable's value after the completion of the forward event handler.

One example of this is when the message passed to an event handler is used as a buffer to state-save original values from an LP's state.
Typically, this is done via a simple `SWAP` macro.
Model developers can run into a very hard to track down bug combining a conditional statement with the SWAP.

These forward and reverse event handlers have the same conditional statements, which at first seem correct, but instead cause the bug:

**Forward Event Handler**:

```
    if (in_msg->value_A == 0) {
        SWAP(lp_state->value_A, in_msg->value_A);
    }
```

**Reverse Event Handler** (*incorrect*):

```
    if (in_msg->value_A == 0) {
        SWAP(lp_state->value_A, in_msg->value_A);
    }
```

The key is to ensure that the same value is checked during the conditional:

**Reverse Event Handler** (*correct*):

```
    if (lp_state->value_A == 0) {
        SWAP(lp_state->value_A, in_msg->value_A);
    }
```

## Bug 2: Wakeup Messages

Self-scheduled wakeup or heartbeat messages are meant to trigger the current LP at a later time.
Typically, they are used when multiple events arrive at the same LP within a close window and, as a group, are meant to trigger a single LP action (this may be an artifact of adding random jitter to prevent tie events).

While it is obvious when to set the flag during forward event handling, the reverse event handler may not *always* want to undo the flag.

### Erroneous Example

**Forward Event Handler**:

```
    if (lp_state->wake_up_sent_flag == 0) {
        lp_state->wake_up_sent_flag = 1;
        tw_event *wake_up_msg = tw_event_new(lp, lp_state->wake_up_time, lp->gid);
        wake_up_msg->type = WAKEUP;
        tw_event_send(wake_up_msg);
    }
```

**Reverse Event Handler** (*incorrect*):

```
    if (lp_state->wake_up_sent_flag == 1) {
        // was this event the one that triggered the wakeup message???
        lp_state->wake_up_sent_flag = 0; // ??????
    }
```

### Corrected Solution

The solution is to use the bit field (defined here as the `tw_bf bitfield` parameter):

**Forward Event Handler** (*augmented*):

```
    bitfield->c1 = 0;
    if (lp_state->wake_up_sent_flag == 0) {
        bitfield->c1 = 1;
        lp_state->wake_up_sent_flag = 1;
        tw_event *wake_up_msg = tw_event_new(lp, lp_state->wake_up_time, lp->gid);
        wake_up_msg->type = WAKEUP;
        tw_event_send(wake_up_msg);
    }
```

**Reverse Event Handler** (*correct*):

```
    if (bitfield->c1 == 1) {
        bitfield->c1 = 0;
        lp_state->wake_up_sent_flag = 0;
    }
```
