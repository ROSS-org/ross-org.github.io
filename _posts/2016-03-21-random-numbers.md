---
layout: post
title: "Random Numbers"
category: feature
---

ROSS's reversible random number generator is based on L'Ecuyer's Combined Linear Congruential Generator (see the [implementation paper](http://www.sciencedirect.com/science/article/pii/S0378475497000529) or the [wikipedia article](https://en.wikipedia.org/wiki/Combined_Linear_Congruential_Generator)).
On top of this implementation, ROSS adds the ability to "rewind" the RNG, a functionality needed for reverse computation.

## The Basics

Each LP has its own stream of random numbers.
This stream is accessed through the `lp->rng` variable.

### Initialization

ROSS includes a default seed for simulations.
By using the same RNG seed for every simulation, ROSS models should be deterministic.
Model developers should not worry about seeding the RNG until the model is fully developed and is deterministic (even in optimistic mode).
Seeding the RNG is covered in the *Advanced Usage* section of this article.

### Generating Random Numbers

The ROSS API provides several random number distribution functions.
Each of these functions take the LP's RNG as an argument (`lp->rng`).

- Uniform Distribution
    - `double tw_rand_unif(tw_rng_stream * g)`
    - returns a double between 0 and 1.
    *Note: this is actually a macro.*

- Integer
    - `long tw_rand_integer(tw_rng_stream * g, long low, long high)`
    - returns a uniform random number between the ranges of `low` to `high` inclusive.

- Binomial Distribution
    - `long tw_rand_binomial(tw_rng_stream *g, long N, double P)`
    - returns the number of successes (i.e., less than probability *P*) of the *N* Bernoulli trials.

- Exponential Distribution
    - `double tw_rand_exponential(tw_rng_stream *g, double Lambda)`
    - returns a exponentially distributed random number with mean, `Lambda`.

- Gamma Distribution
    - `double tw_rand_gamma(tw_rng_stream *g, double shape, double scale)`
    - returns a Gamma distributed random number of a particular shape and scale.
    **WARNING: may need to be reversed more than once!**

- Geometric Distribution
    - `long tw_rand_geometric(tw_rng_stream *g, double P)`
    - returns the number of trials up to and including the first success (i.e., less than probability `P`).

- Default Normal Distribution
    - `double tw_rand_normal01(tw_rng_stream *g)`
    - returns a *unit* or standard normal distributed random number, where the mean and standard deviation of the distribution are 0 and 1 respectively.
    **WARNING: may need to be reversed more than once!**

- Adjustable Normal Distribution
    - `double tw_rand_normal_sd(tw_rng_stream *g, double Mu, double Sd)`
    - returns a normal distributed random number where the mean is `Mu` and the standard deviation is `Sd`.

- Pareto Distribution
    - `double tw_rand_pareto(tw_rng_stream *g, double scale, double shape)`
    - returns a Pareto distributed random number of a particular shape.
    The `scale` parameter is used to adjust it to fit a particular mean.
    The mean of the distribution is *shape/shape-1*.

- Poisson Distribution
    - `long tw_rand_poisson(tw_rng_stream *g, double Lambda)`
    - returns the number of arrivals in a given time period *t* assuming some average arrival rate, *Lambda*.
    **WARNING: may need to be reverse more than once!**


### Reversing

As of [b5235de113](https://github.com/carothersc/ROSS/commit/b5235de113a34788d7f00f6b7d1e831ff3f63cd6) each RNG has a member holding the number of random numbers it has generated.
This is required for delta encoding and should be used by model developers to reverse their RNG calls correctly.

### Example

Each message struct should contain a variable `long rng_count` which will be used to save the number of random numbers generated during forward event execution.
Each event handler should start with the following:

```
    // Put this at the top of your event handler
    long start_count = lp->rng->count;
```

Every exit from the event handler should include (think about using a `goto` statement if the model logic is complicated):

```
    // Put this near (any) function exit
    msg->rng_count = lp->rng->count - start_count;
```

To reset the LP's rng during a reverse event, use the following code snippet:

```
    // Put this in your reverse event handler
    long count = msg->rng_count;
    while (count--) {
            tw_rand_reverse_unif(lp->rng);
    }
```

### Warning

If you need to use a random number to determine a branch during forward execution, you may need to use the same random number to determine a branch during reverse execution.
In this situation, the basic method described here won't help...
It is recommended that you try using a bit field variable.
Another option for reversing the RNG is described in the *Advanced Usage* section.

## Advanced Usage

### Multiple RNGs per LP

If one LP encapsulates more than one statistically independent actions, it should use than one random number stream.
This is done through the `unsigned int g_tw_nRNG_per_lp` global variable.
This variable should be identically set on each MPI-rank before the call to `tw_define_lps` to ensure proper seeding.
*The default value is 1.*

### Seeding

The base RNG needs a single seed, one that should be identically set on each MPI-rank (see the example below which is taken from the [rng model](http://github.com/carothersc/ROSS-Models/tree/master/rng)).
This seed sets the "0-spot" for the simulation within the RNGs period of 2^121 values.
By using the same seed in two different simulations, ROSS models are deterministic.

Each LP within a simulation gets its own stream of random numbers (or multiple streams if needed).
We do this through *seed-jumping*.
Starting at the 0-spot for the simulation, each individual LP RNG make jumps ahead in the overall RNG stream, each jump is approximately 2^70 calls apart.
The number of jumps ahead is determined by the LP ID and the g_tw_nRNG_per_lp variable.
The seed-jumping technique is extremely fast, much faster than reading a file of pre-generated seeds.
Additionally, when you want to have 100K or even 1 million LP simulations, such seed files are not very practical.

#### Using a Different Seed

After a model is fully developed and deterministic, one may want to do experiments with a different seed.
This is used to run the same experiment with different random number streams.
The results of several experiments with different seeds should be combined in a statistical average.

#### Seed Specification

A simulation seed is set by 4, 32-bit values.

ROSS's default seed is `int32_t seed[4] = { 11111111, 22222222, 33333333, 44444444 }`.

All valid seeds, specified by `s = { s1, s2, s3, s4 }`, are defined such that:

```
    1 <= s1 <= 2147483646
    1 <= s2 <= 2147483542
    1 <= s3 <= 2147483422
    1 <= s4 <= 2147483322
```

#### Example

ROSS includes a default seed, thus a model developer does not need to do anything to seed the RNG.

When a simulation requires a different RNG stream, the model can change the initial simulation seed.
To change the seed, a model must set `g_tw_rng_seed` before the call to `tw_init`, like so:

```
    int model_main (int argc, char *argv[]) {
        int32_t local_seed[] = { 5555, 6666, 7777, 8888 };
        g_tw_rng_seed = local_seed;
        tw_init(&argc, &argv);
        ...
```

### Details on Random Number Generation and Reversal

This section describes, in detail, the implementation of ROSS's RNG.
These functions are **NOT** part of the API and do not need to be understood by the typical model developer.

#### Generation

The API functions for generating random numbers in a distribution (described above) each make one or more calls to the ROSS API marco `double tw_rand_unif(tw_rng_stream *g)` which wraps the CLCG-4 function `double rng_gen_val(tw_rng_stream *g)`.

It is important to note that some ROSS random distribution functions make multiple internal calls to the `rng_gen_val()` function.

#### Reversal

The reverse of `rng_gen_val` is the function `double rng_gen_reverse_val(tw_rng_stream * g)`, which is wrapped by the ROSS API macro `double tw_rand_reverse_unif(tw_rng_stream *g)`.
This routine is used to reverse the RNG stream exactly one call.
Thus, for distributions that make multiple calls to the RNG, you will need to save that information and call this function that many times in order to restore the RNG seeds to the state prior to the execution of this event.

It is possible for a model developer to "undo" their own forward calls to `tw_rand_unif` with reverse calls of `tw_rand_reverse_unif`.
This can be helpful in models with a complicated control flow, or with a branch determined by a random number.
In fact, this used to be only way of doing so and several models still use this method.
The only caveat is that some ROSS API random functions call `tw_rand_unif` more than once and thus may require multiple `tw_rand_reverse_unif` to undo.
