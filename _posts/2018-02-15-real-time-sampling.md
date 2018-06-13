---
layout: post
title:  "Real Time Sampling"
author: Caitlin Ross
category: instrumentation
---

This collects data at real time intervals specified by the user.
The runtime option `--rt-interval=n` sets the sampling interval, where n is the number of milliseconds between sampling points.
The default is set to 1000 ms.

### Simulation engine data sampling
To turn on simulation engine sampling, use the option `--engine-stats=n`, where n = 1 for GVT-based sampling, 2 for real time sampling, 3 for virtual time sampling, or 4 to turn on all modes.
No model code modifications are needed to collect simulation engine data for any instrumentation mode. 

### Model-level data sampling
To turn on the model-level data sampling, you need to use `--model-stats=n`.
You can turn it on for the GVT-based sampling (`--model-stats=1`), real time sampling (`--model-stats=2`), virtual time sampling (`--model-stats=3`), or all modes (`--model-stats=4`).
It will sample the model data at the same time as the simulation engine data, depending on the modes you have turned on.
You do not actually need to turn on the simulation engine instrumentation though.
For instance, setting `--model-stats=1` with no other ROSS instrumentation options specified, will sample model data at each GVT, but will not sample the simulation engine data.


