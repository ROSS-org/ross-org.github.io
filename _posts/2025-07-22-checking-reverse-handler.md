---
layout: post
title:  Checking Reverse Handler - The debugging tool for PDES you didn't know you needed
author: Elkin Cruz
category: feature
---

We have implemented a new synchronizaton algorithm: sequential rollback check (`--synch=6`).

As with the optimistic debug mode (`--synch=4`), this mode is not intended for production, but instead it is meant to aid debugging and simplify the life of the developer.

This mode allows you to **test if all event reverse handlers have been properly implemented**.

It will run each event twice, rollbacking it once, and it will check that:

1. Processing and rollbacking an event will produce the same LP state as it would be NOT processing the event at all, and
2. Processing the event after rollbacking is the same as just processing the event once.

Additionally to checking the LP state, we also check the state of random generators.

The runtime penalty of processing all events twice is an increase of at least 100%. In phold experiments, this mode takes about 2.8x times the original.

## Implement with complex models

As it can be seen in the example [phold](https://github.com/ROSS-org/ROSS/blob/f7c413afe4067c5c965f174076d4100be7356d8a/models/phold/phold.main.c#L168) model, we can optionally implement some functions that allows us to print the internal state of the LPs and events. We can also compare LP types that point to other parts of memory (like `malloc`'ed data), i.e., we need deep-copy and deep-check functions for complex LP types.

You can to implement up to six functions (and a struct). You can implement only the functions you need:

- `print_lp(FILE * out, char const * prefix, struct your_lp_type * state)`: this function will take your LP and will print it on the screen (to `out`). To allow this function to print nicely when there are LPs within LPs (as it is the case of [CODES](https://github.com/codes-org/codes)), you have to print at the start of every new line the `prefix` to the line.
- `save_lp(struct your_lp_type_checkpoint * into, struct your_lp_type const * from)`: allows you to copy what is important for your LP state from `from` and into `into`. `your_lp_type_checkpoint` can be the same `your_lp_type`. If you define this function, you might need to implement `clean_lp` and `check_lps`!
- `clean_lp(struct your_lp_type_checkpoint * into)`: companion to `save_lp`; it should free and clean anything that needs cleaning within `into`.
- `check_lps(struct your_lp_type_checkpoint * before, struct your_lp_type * after)`: will determine if the LP stored in memory and the current LP are the same. If you are not checking something with this function, or if there is an issue with this function, the check might fail or pass silently. Fortunately, building this function is as straightforward as printing the state of the LP.
- `print_checkpoint(FILE * out, char const * prefix, struct your_lp_type_checkpoint * state)`: equivalent of `print_lp` for `struct your_lp_type_checkpoint`. If you are using the same LP struct for the checkpoint, you can reuse this function too.
- `print_event(FILE * out, char const * prefix, struct your_lp_type * state, struct your_message * message)`: use this function to print the state of the event the LP was processing when it failed.

The following is a complete (although very simple) implementation of all the functions mentioned (taken from the current phold implementation):

```c
struct phold_state_checkpoint {
    long int saved_dummy_data;
};

void save_state(struct phold_state_checkpoint * into, struct phold_state const * from) {
    into->saved_dummy_data = from->dummy_state;
}

void clean_state(struct phold_state_checkpoint * into) {
    // Nothing to do
}

void print_state(FILE * out, char const * prefix, struct phold_state * state) {
    fprintf(out, "%sstruct phold_state {\n  dummy_state = %ld\n}\n", prefix, state->dummy_state);
}

void print_state_saved(FILE * out, char const * prefix, struct phold_state_checkpoint * state) {
    fprintf(out, "%sstruct phold_state_checkpoint {\n  saved_dummy_data = %ld\n}\n", prefix, state->saved_dummy_data);
}

void print_event(FILE * out, char const * prefix, struct phold_state * state, struct phold_message * message) {
    fprintf(out, "%sstruct phold_message {\n  dummy_data = %ld\n}\n", prefix, message->dummy_data);
}

bool check_state(struct phold_state_checkpoint * before, struct phold_state * after) {
    return before->dummy_state == after->dummy_state;
}

```

Once you have defined all the functions, you have to connect them to their associated LP. For this, do something like:

```c
crv_checkpointer phold_chkptr = {
    .lptype = &mylps[0],
    .sz_storage = sizeof(struct phold_state_checkpoint),
    .save_lp = (save_checkpoint_state_f) save_state,
    .clean_lp = (clean_checkpoint_state_f) clean_state,
    .check_lps = (check_states_f) check_state,
    .print_lp = (print_lpstate_f) print_state,
    .print_checkpoint = (print_checkpoint_state_f) print_state_saved,
    .print_event = (print_event_f) print_event,
};
crv_add_custom_state_checkpoint(&phold_chkptr);
```

Happy debugging!!
