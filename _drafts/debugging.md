---
layout: post
title:  Debugging Tips & Tricks
author: Elsa Gonsiorowski
category: model-dev
---

Debugging a PDES model can be a very tricky venture.
Fortunately, there seem to be a small subset of common bugs that are found in many different models.
This post discusses the basics of debugging a parallel program and some of the bugs to watch out for.
Learn from mistakes of the past!

*If you have run across any bugs which you think should be included here,* **Please** *write up a brief description of your bug and include it in a [pull request]() or [GitHub issue]().*

## When is a Model Correct?

Any valid PDES model should have one simple property: <mark>determinism</mark>.
That is, performing the same simulation using different synchronization algorithms or parallel configurations should yield the same <mark>net event count</mark>.

Typically, the model development process starts by testing a model without  sequential

## Debugging in Parallel

### Print statements

Parallel programs can be large and complex.
Despite this fact, print statements are the quickest and easiest way to gain some understanding of what is happening during execution.
Often, when debugging event handlers, you should print the LP's GID and the event type it is currently processing.

### GDB

### XCode and other IDEs


## Bug 1: Conditionals with SWAP

It is common to use the message passed to an event handler as a buffer to state-save original values from an LP's state.
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
