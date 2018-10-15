---
layout: post
title:  "Streaming Data with Damaris"
author: Caitlin Ross
category: instrumentation
---

*Note*: This is still under heavy development.

To use, you'll need to check out the `streaming-plugin` branches of both ROSS and ROSS-Damaris (same for CODES, if using that).
Right now you can stream simulation engine data in either the GVT-based or Real time sampling modes. 
Virtual Time sampling for simulation engine data will be available after the development of data reduction modules for the in situ analysis system.
Model data can now be streamed in any of the 3 sampling modes (as a reminder, it is only guaranteed to be causally correct in Virtual Time sampling mode).
Event tracing data is not yet supported with Damaris Streaming. 
As a side note, I wouldn't mix and match instrumentation modes just yet (so don't use more than one different mode, and use the same mode if you want to stream both types of data). 

#### Build
Follow the build instructions on the [ROSS-Damaris Overview page](insitu-vis.html).
There's no need to follow the instructions for the section on enabling VisIt support.

You will need a [FlatBuffers](http://google.github.io/flatbuffers/) install for this branch.
When building ROSS with Damaris support, you'll need to set the cmake option `FLATBUF_DIR` to your install directory.

Also you'll need to add the path to libdamaris.a as well as the your ROSS/build/install/lib directory to `LD_LIBRARY_PATH`.

#### Running
There's a new configuration option being introduced in this branch.
Instead of setting all of the configuration options on the command line,
you should use the configuration file.
This file is only for Instrumentation settings and certain Damaris settings.

So essentially when you run your simulation you'll do something like:
```C
$ mpirun -np 5 ./phold --synch=3 (other ROSS/model options) --enable-damaris=1 \
--data-xml=path/to/xml/file --config-file=path/to/config-file
```

For the XML file, you can use the `template.xml` file available in the ROSS-Damaris repo for using this with simulation engine data.
This is the metadata file necessary for Damaris.

For the other configuration file, you can use the `example-config.cfg` file in the ROSS-Damaris repo.
An example of this file is:
```C
[inst]
engine-stats = 1
model-stats = 1
num-gvt = 100
rt-interval = 1000.0
vt-interval = 500
vt-samp-end = 10000
pe-data = true
kp-data = true
lp-data = true
[damaris]
write-data = false
stream-data = true
[damaris.stream]
address = <IP address>
port = <port>
```

The first section `[inst]` is for instrumentation settings. It's similar to how the command line options are currently for the ROSS instrumentation.
For `engine-stats` and `model-stats`, 1 is for GVT-based sampling, 2 is for real time sampling, and 3 is for virtual time sampling, and 0 means no sampling at all.  
So for the example here, it would run GVT-based instrumentation, sampling every 100 GVT computations
and collect PE, KP, and LP level data.
The `write-data` option is a testing feature. This lets the Damaris server write out the flatbuffers
that it's going to stream.
`stream-data` of course turns on streaming capabilities, which will need you to set the `address` and the `port`.


