---
layout: post
title:  "Step 1: Designing Objects"
author: Elsa Gonsiorowski
category: walkthrough
---

The very first thing any ROSS user should do is develop their own model.
This is the best way to gain experience with ROSS API and the best way to expose oneself to the principles of discrete-event simulations.
Here we start with a definition of discrete-event simulation and begin to create a basic model for airports.

## Discrete-Event Simulations

Computer simulations model real world behavior in order to understand a complex system over time.
Within discrete-event simulations the state of the modeled system can only be modified through a sequence of time-stamped events [6]. Thus these simulations are constructed through two entities:

- Events:
    Each event occurs at a particular instance in time and contains a specific piece of information.
    An event will directly affect only one logical process, but a series of events can be said to be causally linked.
- *Logical Processes* (LPs):
    These entities encapsulate the particular state of a portion of the modeled system.
    LPs are directly affected by events, and can cause change to other parts of the system by “sending” or “scheduling” events in the future.

## Defining the Airport Model

With the basic understanding that our simulations are made up of LPs and events, we can begin to define how a model for airports might look.
Let's assume that we want to understand the way in which unexpected weather delays affect congestion on an airport runway.
To start, we should create a model of airplanes traveling between airports.

### LPs in the System

First there are two basic objects in an airport system:

- The airport itself;
- The airplanes that travel between airports.

As in a traditional object-oriented programming approach, we can begin by diagramming the state variables for each object.
These will each be represented as a distinct LP-types in ROSS.

     Airplane                                            Airport
    ----------                                          ---------
    - current status (on the ground or in flight)       - status of the runway (free or busy)
    - current airport                                   - list of airplanes waiting to land
    - destination airport                               - list of airplanes waiting to take off

***Best Practice***:
Always start with a bare-bones model.
Simulations are an approximation of the real world and there is often no need to model the evolution of non-relevant details.
For example, this airport-runway model does not need to keep track of airplanes at the gate or how cargo is loaded and unloaded from a plane.
Also, when the simulations become extremely large, memory can be a limiting factor.

### Events in the System

With these two types of objects, airports and airplanes, there are some simple messages that should pass between them for landing an take-off:

1. An airplane makes a request to use the runway at the airport
2. An airport grants an airplane access to its runway
3. An airplane has completed its landing or take-off (and thus relinquishes use of the runway)

These simple messages do exchange any additional pieces of information so there is no need for associated variables.

***Limitation***:
For enhanced simulation performance, ROSS requires all events take up the same size in memory.
Thus, there is only one `message` object.
This object should include a `type` variable to distinguishing it during event processing.

## Coding the Airport Model

Since the LP and events objects are essential to your model, they should be defined within a header file.
This is also the place to include the `ross.h` library.
Code for this step can be found within the [model.h file of airport-model](https://github.com/gonsie/airport-walkthrough/blob/28a0b587c1488f6937d98fa6b5ebd6377a5870c7/model.h) repository.

### LP State

The LP objects in ROSS are comprised of more than just the airplane or airport state variables.
They include per-LP random number generators and other book-keeping variables.
Thus, it important to understand there is a difference between the full LP object and the LP state.
We say that *an LP object contains an LP-state*.

The header file for our model simply defines the state structs for the LPs in our simulation.
These structs hold define the variables that define the state of an airplane or airport.
They are called `airplane_state` and `airport_state` respectively.

### References to LPs: `tw_lpid`

Since ROSS simulation can be distributed, traditional C pointers to LP objects are futile.
Within ROSS we can use the `tw_lpid` type (usually defined as `unsigned int`).
These IDs are a unique way to reference an individual LP in the simulation.

### C Coding Style

The airplane model code takes advantage of some the syntax shortcuts within the C language.
First we use a typedef shortcut:

- `typedef struct {...} struct_type_name;`
- `typedef enum {...} enum_type_name;`

Second, we use ALL CAPS for the names of our type-defined values.

### Data Structures: Lists/Arrays

The model exists within the ROSS framework and, as such, should notify ROSS of any memory allocations it would like.
This makes it hard to use any existing libraries (such as boost or the STL) for higher-level data structures.
While C does allows arrays to exist within a struct, there are some limitations.

For now, we have simply left the lists for airplanes waiting to land and airplanes waiting to take off as `tw_lpid` pointers.
The implementation of these data structures will be discussed later.

### Events vs. Messages

Similar to the subtle difference between an LP and an LP-state discussed above, ROSS distinguishes betweens events and messages in the following way: *an event encapsulates a message*.
Thus, our model defines the `message` struct.

Again, it is important to point out that there is, and can only be, one message struct.
All events (and thus all messages) in a ROSS simulation must be the same size in memory.
