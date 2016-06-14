---
layout: post
title:  "Step 2a: Modeling Airplane Functionality"
author: Elsa Gonsiorowski
category: walkthrough
---

Now that the Airplane and Airport LPs have been defined, their functionality can be implemented.
During the model implementation process, there are several non-trivial design decisions that must made.
This post attempts to document thought process and order of steps that takes place during the implementation of the airplane functionality.

## Initialization

Before a simulation can start there are two things must be done:

1. Initialize each LP state
2. Create and schedule the initial simulation events

There are two callbacks available for this purpose: `init_f` and `pre_run_f`.

### Airport LP State

The variables that make up the state of an airplane LP are:

- status
- current airport
- destination airport

These state variables are initialized in the `init_f` callback.

The initialization of the status variable is simple: each airplane begins on the ground.
The implementation of the current and destination airport variables is a little trickier.
The simplest way to initialize these variables is to set them to random values.
The only information needed is the total number of airports in the simulation.

### Global Variables

Global variables for a model should be declared (using `extern`) in the main header file and defined an appropriate C file.
For this model, there are two global variables:

- total number of airports in the simulation,
- total number of airplanes in the simulation.

For now, we will also specify some *magic numbers* for an airplane.
These are implemented as `#define` statements in `airplane.c`.
The magic numbers include:

- time it takes for a airplane to use a runway,
- time an airplane spends in flight,
- time an airplane spends on the ground.

This modeled behavior could be more true-to-life but, for now, we will use these constant values.

### Initial Simulation Events

Since ROSS is an engine for *event driven* simulation, there must be scheduled events in order for simulation to progress.
Thus, to start simulation, there must be a set of kickoff events.
Note, any events scheduled within the `init_f` function must be self-scheduled.
To schedule initial events to a different LP use the `pre_run_f` callback.

For this model, the initial events are the airplanes requesting to use the runway to take off from their current airports.
The timing of this event requires a random number.
Thus, we use the `tw_random_unif` function, which returns a uniform random value between 0 and 1 (inclusive).
This random value, times the constant `GROUND_TIME`, determines when the airplane is ready to request the runway.

## Event Processing

Since events are the drivers of change in these simulations, processing them can become tricky.
The initial goal of model development is to create a logical, functioning forward event handler.
This function is triggered by an event and causes both state changes in an LP as well as the scheduling of other events.

### Forward Event Handler

For airplanes, there in only one type of messages they should receive: a runway grant.
The series of actions that is trigged from a runway grant is similar in both the landing and takeoff cases:

<div class="col2">
**Take Off**

1. Use runway for takeoff
2. Exist in flight
3. Request runway at destination

{: .rc}
**Landing**

1. Use runway for landing
2. Exist on the ground
3. Request runway for takeoff

</div>

Note that at some point, the airplane's `destination_airport` becomes its `current_airport`.

### Reverse Event Handler

As noted above, the implementation of the reverse event handler should be left until after correct serial and conservative simulations have taken place.
One thing to remember while programming the *forward* event handler is that the reverse event handler will have a very similar control flow.
Simple and uncomplicated forward event handlers will pay off in the long run.

For now, a simple `return` is all that is needed for the reverse event handler.

## Finalize

The LP finalize function is most often used for collecting and printing per-LP statistics.
For now, this function simply returns.
