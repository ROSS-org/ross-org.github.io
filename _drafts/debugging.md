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

| Same | Different |
|------+-----------|
| Simulated amount of time <br /> LPs <br /> random number generator seeds | Synchronization algorithms <br /> parallel configurations |

Typically, the model development process starts by testing a model without  sequential

## Debugging in Parallel

### Print statements

Parallel programs can be large and complex.
Despite this fact, print statements are the quickest and easiest way to gain some understanding of what is happening during execution.
Often, when debugging event handlers, you should print the LP's GID and the event type it is currently processing.

### GDB

### XCode and other IDEs


## Bug 1: Conditionals with SWAP

{::options parse_block_html="true" /}
<div class="col2">
Forward Event Handler:

```
if (in_msg->value_A == 0) {
	SWAP(lp_state->value_A, in_msg->value_A);
}
```

{: .rc}
Reverse Event Handler (*incorrect*):

```
if (in_msg->value_A == 0) {
	SWAP(lp_state->value_A, in_msg->value_A);
}
```
</div>

<div class="col2">
Forward Event Handler:

```
if (in_msg->value_A == 0) {
	SWAP(lp_state->value_A, in_msg->value_A);
}
```

{: .rc}
Reverse Event Handler (*correct*):

```
if (lp_state->value_A == 0) {
	SWAP(lp_state->value_A, in_msg->value_A);
}
```
</div>

## Bug 2: Wakeup Messages

Forward Event Handler:

```
if (lp_state->wake_up_sent_flag == 0) {
	lp_state->wake_up_sent_flag = 1;
	tw_event *wake_up_msg = tw_event_new(lp, lp_state->wake_up_time, lp->gid);
	wake_up_msg->type = WAKEUP;
	tw_event_send(wake_up_msg);
}
```

Reverse Event Handler (*incorrect*):

```
if (lp_state->wake_up_sent_flag == 1) {
	// was this event the one that triggered the sending of a wakeup message???
	lp_state->wake_up_sent_flag = 0; // ??????
}
```
