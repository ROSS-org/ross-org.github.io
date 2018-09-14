---
layout: post
title:  "In Situ Analysis and Visualization with Damaris"
author: Caitlin Ross
category: instrumentation
---

[Damaris](https://project.inria.fr/damaris) is an I/O and data management software.
Support for Damaris is currently being added to ROSS to enable in situ data analysis and visualization. 
The current focus is to use it with the various instrumentation modes to do performance analysis on simulations to better understand performance bottlenecks.
Eventually it will support visualizing model data as well.

ROSS is currently up to date with [Damaris version 1.2.0](https://project.inria.fr/damaris/download).

## Building ROSS with Damaris

#### Install Damaris.
For Damaris installation instructions, see the [Damaris User Guide](https://project.inria.fr/damaris/documentation).
I followed the recommended instructions, which is to install the dependencies in `$HOME/local`.

*Note*: I did have some issues due to Boost.
If you have multiple versions of Boost installed, make sure the build process is only trying to use headers/libraries from a single version.

#### Get ROSS-Damaris submodule.
Using Damaris with ROSS relies on the [ROSS-Damaris](https://github.com/caitlinross/ROSS-damaris) repo, which is also a submodule in the ROSS repo.
In your clone of the ROSS repo, you can do the following to get the Damaris component:

```C
$ git submodule init damaris
$ git submodule update damaris
```

This should create the `damaris` directory in `ROSS/core`.

#### Build ROSS with Damaris support.
Set `ARCH` and `CC` as usual with a ROSS build.
You'll also need to set `CXX`.
Now run ccmake (or use cmake and make sure to set the appropriate arguments).
You should set `CMAKE_BUILD_TYPE` and `CMAKE_INSTALL_PREFIX` as you wish.
Set `USE_DAMARIS` to `ON` and set `DAMARIS_DIR` appropriately for Damaris and its dependencies (e.g., `$HOME/local`).
At this point, if you're using ccmake, it will pop up the `USE_VISIT` and if that is turned on, then `VISIT_DIR`.
See below for more details on using VisIt.
Now configure and generate the files as usual with ccmake.
Finally `make` and `make install` and you should be good!

Cmake will recognize `DAMARIS_DIR` and `VISIT_DIR` as environment variables, so you could set them in your .bashrc or something similar, if desired.

For linking with your model, you will also need to set `LD_LIBRARY_PATH` to your Damaris install.
So if you followed the recommended Damaris installation instructions, you'd set it to `$HOME/local/lib`.



## Setting up and running your model with ROSS-Damaris

#### Build configuration
Next, you'll need to do some setup with your model to make sure it will correctly link with Damaris and its dependencies.
So far this has been tested with PHOLD, as well as some CODES network models.  
The basic instructions for setting up with another ROSS model (any non-CODES model) should be the following:

Add the following commands to your `CMakeLists.txt` file in your model directory.
```C
IF(USE_DAMARIS)
  INCLUDE_DIRECTORIES(${DAMARIS_INCLUDE})
ENDIF(USE_DAMARIS)
```

After you've created the executable for your model in `CMakeLists.txt` (e.g., phold uses `ADD_EXECUTABLE`), you'll need to add the `ROSS_Damaris` library in your `TARGET_LINK_LIBRARIES` call, like below (replacing `phold` with the name of your model executable):
```C
IF(USE_DAMARIS)
  TARGET_LINK_LIBRARIES(phold ROSS ROSS_Damaris m)
ENDIF(USE_DAMARIS)
```

As long as you've set `USE_DAMARIS` and `${DAMARIS_DIR}`, the `${DAMARIS_INCLUDE}` and `${DAMARIS_LINKER_FLAGS}` will automatically populate with the correct values.

Now you should be able to link your model with ROSS and Damaris.
You will also need to add the path of the `librdplugins.so` library to your `LD_LIBRARY_PATH`, so Damaris can find it. 
This library should be installed to `/path/to/ross-build/install/lib` directory when you do `make install`.

#### CODES models
You can ignore the previous build section for CODES models.
When you do `make install` for ROSS-Damaris, the necessary library file and pkg-config files are installed in the same directory alongside the ROSS library and pkg-config files.
When doing the `./configure` step with CODES, you just need to add `--with-damaris`.
No paths need to be supplied as ROSS puts that info in the pkg-config and the CODES build system will automatically get the necessary CFLAGS, LIBS, etc. 

#### Model Code Changes Needed
In your model's call to `tw_init()`, ROSS will make a call to Damaris that will split the MPI ranks into two subcommunicators, and identify ranks as either ROSS ranks or Damaris ranks.
ROSS ranks make use of the `MPI_COMM_ROSS` subcommunicator.
You need to make sure that Damaris ranks will not enter `tw_run()`.
The call to `tw_end()` will clean up things with Damaris and MPI, so all ranks should call `tw_end()`.
To do this you can check the global variable `g_st_ross_rank`, which is set to 1 if it is a ROSS rank and 0 if it is a Damaris rank.
You should also put any Damaris specific code within preprocessor statements checking if `USE_DAMARIS` is defined.

Here is what the relevant sections of the PHOLD `main()` looks like:
```C
int main(int argc, char **argv, char **env)
{
    // ... ommitted
    tw_opt_add(app_opt);
    tw_init(&argc, &argv);
#ifdef USE_DAMARIS
    if (g_st_ross_rank)
    {
#endif
        if (lookahead > 1.0)

        // ... ommitted

        tw_run();
#ifdef USE_DAMARIS
    } // end if (g_st_ross_rank)
#endif
    tw_end();
    return 0;
}
```

*Warning*: If for some reason you need to use MPI calls in your model, you should NOT use `MPI_COMM_WORLD`.
`MPI_COMM_ROSS` must be used or your simulation will probably deadlock.

#### Running your model with Damaris
To run with Damaris enabled, you'll need to use `--enable-damaris=1`.
There is also an XML file that describes the data being collected to Damaris.
You can create your own, but we recommend using the provided file in the ROSS-Damaris repo, `test.xml`. 
To let ROSS know the location of this file, use the `--data-xml` command line option.
For example:
```C
$ mpirun -np 4 ./phold --synch=3 --enable-damaris=1 --data-xml=/path/to/ROSS-damaris/test.xml`
```
Damaris is configured to run in dedicated-core mode, so for this example run on a single node system, 1 rank will be dedicated to Damaris and 3 ranks dedicated to run ROSS.
So far this has only been tested on a single node, but if you run on more than one node, 1 rank/node should be dedicated to Damaris and the remaining ranks will be dedicated to ROSS.

In this instance, Damaris just runs alongside ROSS, but does nothing interesting.
Right now, Damaris is configured to work with the simulation engine data (at all levels - PE, KP, and LP) for the GVT-based and real time instrumentation modes.
Just use the normal commands for turning on these instrumentation modes and Damaris will take over and data will no longer be written to file.
See the [ROSS Instrumentation Documentation](http://carothersc.github.io/ROSS/instrumentation/instrumentation.html) for instructions on using the instrumenation.
This will be updated as work progresses on integrating with other instrumentation modes and model data.

## Setting up ROSS/Damaris with VisIt
My current setup is running simulations on a remote system, so I had to install VisIt on that system as well as my local system (Note: they need to be the same version).
We're using VisIt version 2.12.1. [VisIt can be downloaded here.](https://wci.llnl.gov/simulation/computer-codes/visit/downloads)
For the local system, I just downloaded the VisIt executable available for my OS.
For the remote system, it's best to download the source so we can change some of the configuration.
You should be able to just download the `build_visit` script for the version you want, which will actually download the VisIt source for you.
This is the configuration I used for building VisIt:
```C
./build_visit2_12_1 --parallel --mesa --server-components-only --makeflags '-j 16'
```
If you're setting up on your local system, you probably want to remove the `--server-components-only` arg.
The build can take a couple of hours because the script downloads and installs dependencies.  

Now that VisIt is set up, you'll actually need to rebuild Damaris.
See the Damaris User Guide for building with VisIt support.
Now you'll have to rebuild ROSS to use VisIt as well.
Once `USE_DAMARIS` is set to on in cmake, you can now set `USE_VISIT`.
In addition, you'll need to set `VISIT_DIR` to the directory containing the VisIt library file.

## Running ROSS with Damaris and VisIt
In the `ROSS-Vis/core/damaris` directory, there is a test.xml file.  This describes the data to Damaris.  The only thing you need to check here is that the path listed for VisIt is correct.  It should look like:
```C
<visit>
    <path>/path/to/visit2.12.1/src</path>
</visit>
```
On your local system, you need to set up a host profile so you can connect to VisIt on the remote system.
Go to Options and then Host Profiles.
Fill out Remote Host name, Path to Visit, Username, and check tunnel data connections through SSH.
Then click launch profiles tab.
Set a profile name and then on the Parallel tab, I checked Launch parallel engine along with setting Parallel launch method to mpirun.
You can then set number of desired processors/nodes.  Click Apply and then Dismiss.

On the VisIt window, click Open under Sources.
Change Host to the Host you just created a profile for.
Enter your password when prompted and the path should change to your home directory on that system.
It will also start the VisIt processes running on the remote system.
Go into the .visit/simulations Directory.
The file list will be empty at this point.
Now you can run your simulation as described above for running with Damaris, but we need to turn on instrumentation (see next paragraph).
Click the refresh button on the File open window and there should be a file that starts with some number followed by `ROSS_test.sim2`.
Click on that and then OK.
Now you can try out different visualizations and choose the variables you want to examine.

## Other notes about using Damaris with ROSS
* Damaris is not yet setup to use the event tracing data yet.
* Same for model data. Model data support should be available soon.
* Using Damaris requires collective calls to end a given iteration, so iterations end at GVT.
Due to this, GVT-based instrumentation works well with Damaris. 
However, real time and virtual time sampling modes can be a little quirky.
There may be iterations where some number of PEs have no data for either of these modes, which appears to be a problem if trying to visualize this data in VisIt.
VisIt will report a problem if it doesn't have data for every generation, though I'm unsure if that's due to VisIt or how Damaris exposes the data to VisIt.
This will be handled appropriately as we develop our own plugins to use in Damaris.
* Setting `--num-gvt` between 100 and 500 has worked well for the PHOLD model when using GVT instrumentation mode.

## Plans for ROSS-Damaris integration
* Use Damaris to stream ROSS data to the various [ROSS/CODES vis tools](https://github.com/HAVEX) that are being developed.
* Have Damaris do more intelligent analysis on the data before passing it off to visualization tools.
* Better reverse computation debugging!
* Integration with 3D network visualizations of CODES models.



