---
layout: post
title: Schedulers
author: Justin LaPre
---

Schedulers are at the core of any parallel discrete-event simulation (PDES) system.
They are responsible for the performance of the simulation and, in some cases, can indirectly influence correctness as well.
Schedulers execute *events* and they do so in *timestamp*, or *virtual time* order.
As each event is executed, time jumps to the timestamp of that event.
In this blog post we will discuss all three major scheduler categories: sequential, conservative, and optimistic.

## Sequential

**Sequential (or serial) simulations continuously execute the next event in timestamp order.**

The sequential (or serial) scheduler is very simple: extract the next event from the priority list, execute that event, repeat.
This scheduler is useful for fleshing out the mechanics of your model with very little concern for performance.
As its name implies, this type of scheduler executes all events in a sequential manner.

While many *logical processes* or LPs may exist for a given simulation, all events are stored in a single priority queue ordered by their virtual time.
In ROSS, each of these queues is contained within a *processing element* or PE
(it is safe to consider each PE as a CPU core for the duration of this post).
Additionally, all LPs are owned by one and only one PE.
In the sequential scheduler, only one PE exists across the entire system.
This ensures that the resulting order of events is necessarily correct, assuming all events are scheduled into the future.

While correct, having only one PE is very limiting.
Multiple PEs is an obvious next step.

## Conservative

**Conservative simulations require that all PEs maintain the same virtual time "window" as they progress throughout the simulation.**

The conservative scheduler is able to utilize more than one PE.
The priority queue of events exist on each of these PEs as well.
In other words, PE<sub>*A*</sub> contains the events *A* must execute; likewise for *B*.
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
Without this constraint, events could spontaneously appear within the lookahead window and disrupt causality.

Very often, conservative simulation yields acceptable performance improvement over the sequential approach.

## Optimistic

**Optimistic simulations allows for widely varying virtual times during execution but must provide a recovery mechanism to undo erroneous events.**

The optimistic (in this case, Time Warp) approach was designed to overcome the limitations of conservative simulation, namely that all LPs must be at approximately the same time.
While allowing virtual time to progress at different rates solves that particular problem, it also introduces another: *straggler events*.
Straggler events arrive with a timestamp *older than the current virtual time* and are therefore out-of-order.
As the name implies, Time Warp must jump back in time, undoing the erroneous event, and restoring the state to its value prior to incorrectly executing it.
At this point, the newly received event can be executed.

There are various mechanisms available to support state recovery.
One approach is copy state saving (CSS) which simply performs an operation to copy the memory (similar to `memcpy()`).
While this approach works, it has the potential to consume large amounts of memory.
Another approach is *reverse computation* whereby operations are reversed.
For example, if we increment a counter (`i++`), that operation can easily be reversed (`i--`).
Not all operations are so simple, however, and some are inherently non-reversible.
In such cases, we must resort to state saving anyway.

Optimistic simulations excel in situations where the lookahead inherent to the model is very small.
In such cases conservative simulations are effectively starved of useful work and event rates are limited.
Increasing lookahead may not always be possible without sacrificing correctness.

Additionally, overly-aggressive Time Warp schedulers can create a *rollback avalanche* whereby one straggler event is able to create a cascading undo events.
These in turn can negatively impact the *efficiency* of the simulation; negative efficiencies are possible and imply that the simulation is performing more rollbacks than "real" work.

## Summary

Moving beyond a single PE requires either conservative or optimistic approaches and each has its drawbacks.
With conservative the simulation must not violate lookahead.
Conversely, optimistic must be cautious about being too aggressive.
There is no free lunch.
