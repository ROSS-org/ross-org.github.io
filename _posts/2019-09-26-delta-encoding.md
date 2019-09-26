---
title: "Delta Encoding"
author: "Justin LaPre"
category: feature
---

Delta encoding provides a solution for models that contain events which are not well suited for using reverse computation or consumes significant amounts of memory (making copy-state approaches infeasible).
Delta encoding solves this issue by only computing state change deltas once an event has completed execution.
These deltas are then compressed for reduced storage overheads.
In addition, delta encoding is done on a per-event basis allowing both reverse computation and delta encoding to be mixed within a single model.
Overall, the delta encoding approach provides the benefits of incremental state-saving but without requiring the specific identification of which state elements change.
This feature is further described in [LaPre et al., 2015](https://dl.acm.org/citation.cfm?id=2888969).

## Use Case

Some model events are not well suited to reverse computation([LaPre et al., 2014](http://dl.acm.org/citation.cfm?id=2601397)).
Two features are often difficult or inefficient to reverse:
- Complex, nested loops,
- Destructive value assignments.

## API

The API is as follows:

    tw_snapshot(lp, lp->type->state_sz)
    tw_snapshot_delta(lp, lp->type->state_sz)
    tw_snapshot_restore(lp, lp->type->state_sz, lp->pe->cur_event->delta_buddy, lp->pe->cur_event->delta_size)

In addition, the following options are provided:

    --buddy-size=n    delta encoding buddy system allocation (2^X) (default 0)
    --lz4-knob=n      LZ4 acceleration factor (higher = faster) (default 17)

### Command-Line Options

The `--buddy-size` option allocates a block of memory for use by the delta encoding system.
This flag **must** be set for all ROSS binaries that use delta encoding.
A value of 20 will allocate 2^20 bytes (1 megabyte) of space and is usually a good starting point.

The `--lz4-knob` option tunes the amount of LZ4 compression which is applied to the delta.
A value of 1 provides maximum compression, but at the cost of speed.
ROSS uses a default value of 17.

### Functions

`tw_snapshot()` and `tw_snapshot_delta()` should be called at the start and end, respectively, of an event handler.
Be careful to ensure that `tw_snapshot_delta()` is called before *every* possible function return.
In the reverse event handler, call `tw_snapshot_restore()` to return the state to its previous values.

## Other Concerns

Random number generation must also be accounted for, which must be done by the model in addition to any delta encoding calls.  If an LP makes four RNG calls, it must make four RNG *uncalls*.  This can be tracked by using the `rng_count` member in the `message` class; see the linked code example for an implementation

## Example

For a complete (and probably working) example, see [the phold-delta model](https://github.com/ROSS-org/ROSS-Models/tree/master/phold-delta).
