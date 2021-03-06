---
layout: post
title:  "Virtual Time Sampling"
author: Caitlin Ross
category: instrumentation
---

In order to support virtual time sampling in ROSS, specialized LPs called Analysis LPs were added to the ROSS core.
These LPs are hidden from the model, so they will not affect any LP to PE/KP mappings.

There is one analysis LP per KP assigned when virtual time sampling is turned on.
This LP is responsible for collecting data from all LPs belonging to the same KP at sampling points.

Turning on virtual time sampling is similar to turning on the other modes: 
To turn on simulation engine sampling, use the option `--engine-stats=n`, where n = 1 for GVT-based sampling, 2 for real time sampling, 3 for virtual time sampling, or 4 to turn on all modes.
To turn it on for the model-level data sampling, you need to use `--model-stats=n`, where n = 1 for GVT-based sampling, 2 for real time sampling, 3 for virtual time sampling, or 4 to turn on all modes.

To set the sampling interval, set `--vt-interval` to the desired value.
The end time for sampling must also be set using the `--vt-samp-end` option. 


#### Other notes
The final model data that is output is causally correct. I.e., there is one sample per data point and it is valid for that time (provided reverse computation is correct).
For simulation engine data, there may be more than one sample per virtual time sampling point for any given entity (i.e., more roll backs, more samples collected).

To read the binary file for virtual time sampling, use the `analysis-lp-reader.py` script located in the [ROSS binary reader repo](https://github.com/caitlinross/ross-binary-reader).
You will need to edit it to allow it to read your model-level data.

#### CODES Specific details
The Fat-tree, Dragonfly (original and custom), and Slimfly network models all have support for the virtual time sampling.
To update these models with your own data collection, these are the functions to edit:

* `slimfly.c`: 
  * `ross_slimfly_sample_fn()` for terminal LPs
  * `ross_slimfly_rsample_fn()` for router LPs
* `dragonfly.c`: 
  * `ross_dragonfly_sample_fn()` for terminal LPs 
  * `ross_dragonfly_rsample_fn()` for router LPs
* `dragonfly-custom.C`: 
  * `ross_custom_dragonfly_sample_fn()` for terminal LPs 
  * `ross_custom_dragonfly_rsample_fn()` for router LPs
* `fattree.c`: 
  * `ross_fattree_sample_fn()` for terminal LPs 
  * `ross_fattree_ssample_fn()` for switch LPs

You can either change these functions or add in different functions and switch the appropriate element in the `st_model_types` declaration to point to your function.
Don't forget to update the size of the data (`sample_struct_sz`) you want ROSS to store at each sampling point in the `st_model_types` struct (see [Instrumentation Overview](instrumentation.html) for more details).

One thing to note is that the value of `sample_struct_sz` for the router/switch LPs in each of these models is set to 0 initially, because they currently collect some data for each port on the router.
The radix is not determined until runtime, since it is a parameter set in the config file, so `sample_struct_sz` is updated to the correct value in the init() functions for their respective models. 
