---
layout: post
title: Schedulers
author: Justin LaPre
---

Schedulers are at the core of any parallel discrete-event simulation (PDES) system.
They are responsible for the performance of the simulation and, in some cases, can indirectly influence correctness as well.
In this blog post we will discuss all three major scheduler categories: sequential, conservative, and optimistic.

## Sequential

The sequential (or serial) scheduler is very simple: extract the next event from the priority list, execute that event, repeat.
This scheduler is useful for fleshing out the mechanics of your model with very little concern for performance.
As its name implies, this type of scheduler executes all events in a sequential manner.

While many *logical processes* or LPs may exist for a given simulation, all events are stored in a single priority queue ordered by their execution time.
In ROSS, each of these queues is contained within a *processing element* or PE
(it is safe to consider each PE as a CPU core for the duration of this post).
Additionally, all LPs are owned by one and only one PE.
In the sequential scheduler, only one PE exists across the entire system.
This ensures that the resulting order of events is necessarily correct, assuming all events are scheduled into the future.

While correct, having only one PE is very limiting.
Multiple PEs is an obvious next step.

## Conservative

The conservative scheduler is able to utilize more than one PE.
The priority queue of events exist on each of these PEs as well.
In other words, if you have a PE *A* and PE *B*, PE<sub>*A*</sub> contains the events *A* must execute; likewise for *B*.
As long as there is no inter-PE exchange of events, both PEs are capable of executing their events independently.
While quite fast, this requirement is also very limiting.

Most simulations need to send events from one LP to another, possibly spanning PE boundaries.
In turn, this requires some degree of coordination or synchronization between all LPs in the system.
As long as each LP executes its events in non-decreasing timestamp order, the simulation will be correct.
This is referred to as the *local causality constraint*.

While the resulting event execution may be correct, it does not prevent deadlock.
See [here](http://www.acm-sigsim-mskr.org/Courseware/Fujimoto/Slides/FujimotoSlides-06-NullMessages.pdf) for a slide set illustrating why.
By utilizing the CMB algorithm described on that link, deadlock is avoided.

An additional piece of information useful in constructing conservative simulations is a value called *lookahead*.
Lookahead refers to the minimum (but non-zero) time delta, from the perspective of the currently-executing event, beyond which future events must be scheduled.
This is necessary to guarantee forward progress is made.
All events within the lookahead window (between *t*<sub>now</sub> and *t<sub>now</sub>+L*, where *L* is the lookahead) can be executed.
Without this constraint, events within the lookahead window could spontaneously appear and disrupt causality.

## Optimistic

- Time Warp
