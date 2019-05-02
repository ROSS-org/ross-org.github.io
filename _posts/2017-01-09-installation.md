---
layout: post
title:  "Manual ROSS Installation"
author: Caitlin Ross
category: setup
---

ROSS can now be installed using the [Spack package manager](https://spack.io/).
To see those instructions, please see [Installing ROSS with Spack](spack.html).

The ROSS code base is easy to install and has been tested on both MacOS and Linux systems.
It is a lightweight, C library with a limited number of dependencies.

## Requirements

1. ROSS is written in C standard and thus requires a C compiler (C11 is prefered, but not required).
2. The build system is [CMake](www.cmake.org), and we require version 3.5 or higher.
3. ROSS relies on MPI. We recommend the [MPICH](www.mpich.org) implementation.

## Manual Build

ROSS can be installed quickly by following these simple steps:

- Clone the repository to your local machine:

```C
$ git clone https://github.com/ROSS-org/ROSS.git
$ cd ROSS
```

- *Optional* Symlink your model to ROSS.
Please see [this blog post](https://github.com/ross-org/ROSS/wiki/Constructing-the-Model) for details about creating and integrating a model with ROSS.

```C
$ ln -s ~/path-to/your-existing-model models/your-model-name
```


- Create a new ross-build directory. ROSS developers typically do out-of-tree builds.

```C
$ cd ~/directory-of-builds/
$ mkdir ross-build
$ cd ross-build
```

- We use [CMake](www.cmake.org) to build ROSS. <br/>
```C
$ ccmake ~/path-to/ROSS
```
  * You'll want to change `CMAKE_INSTALL_PREFIX` to the directory you want ROSS installation files, e.g., `$HOME/ross-build/install`
  * If you want to use the PHOLD model, you'll want to set `ROSS_BUILD_MODELS` to ON.
  * You may also want to set `CMAKE_BUILD_TYPE` to Debug.
  * If you want a static library, no changes are needed. But if you'd like to build a shared library, you should set `ROSS_BUILD_SHARED_LIBS` to ON.

- Finally, we can build:

```C
$ make
$ make install
```

For the make command, some other options are:

```C
$ make -k         // ignore errors from other models
$ make -j 12      // parallel build
$ make model-name // build only one model
```

## Run your model

See [this wiki page](https://github.com/carothersc/ROSS/wiki/Running-the-Simulator) for details about the ROSS command line options.

```C
$ cd ~/directory-of-builds/ROSS-build/models/your-model
$ ./your-model --synch=1               // sequential mode
$ mpirun -np 2 ./your-model --synch=2  // conservative mode
$ mpirun -np 2 ./your-model --synch=3  // optimistic mode
$ ./your-model --synch=4               // optimistic debug mode (note: not a parallel execution!)
$ mpirun -np 2 ./your-model --synch=5  // realtime optimistic mode
```
