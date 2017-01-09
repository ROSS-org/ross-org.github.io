---
layout: post
title:  "ROSS Installation"
author: Caitlin Ross
category: setup
---

## Requirements

1. ROSS is written in C standard and thus requires a C compiler (C11 is prefered, but not required).
2. The build system is [CMake](www.cmake.org), and we require version 2.8 or higher.
3. ROSS relies on MPI. We recommend the [MPICH](www.mpich.org) implementation.

## Build 

ROSS can be installed quickly by following these simple steps:

- Clone the repository to your local machine:

```
$ git clone https://github.com/carothersc/ROSS.git
$ cd ROSS
```

- *Optional* Installing the submodules. Currently, ROSS includes three submodules:
  - [ROSS-Models](http://github.com/carothersc/ROSS-Models) is a set of existing models
  - [template-model](http://github.com/nmcglohon/template-model) is a starting place for new models
  - [RIO](http://github.com/gonsie/RIO) is a checkpointing framework

```
$ git submodule init
$ git submodule update
```

- *Optional* Symlink your model to ROSS.
Please [this wiki page](https://github.com/carothersc/ROSS/wiki/Constructing-the-Model) for details about creating and integrating a model with ROSS.

```
$ ln -s ~/path-to/your-existing-model models/your-model-name
```


- Create a new ross-build directory. ROSS developers typically do out-of-tree builds.

```
$ cd ~/directory-of-builds/
$ mkdir ross-build
$ cd ross-build
```

- Set the following variables:

```
$ export ARCH=x86_64
$ export CC=mpicc
```

- We use [CMake](www.cmake.org) to build ROSS. <br/>
```
$ ccmake ~/path-to/ROSS
```
  * You'll want to change `CMAKE_INSTALL_PREFIX` to the directory you want ROSS installation files, e.g., `$HOME/ross-build/install`
  * If you're using one of the ROSS models, you'll want to set `ROSS_BUILD_MODELS` to ON.
  * You may also want to set `CMAKE_BUILD_TYPE` to Debug.

- Finally, we can build:

```
$ make
$ make install
```

For the make command, some other options are:

```
 make -k         // ignore errors from other models
 make -j 12      // parallel build
 make model-name // build only one model
```

## Run your model

See [this wiki page](https://github.com/carothersc/ROSS/wiki/Running-the-Simulator) for details about the ROSS command line options.

```
$ cd ~/directory-of-builds/ROSS-build/models/your-model
$ ./your-model --synch=1               // sequential mode
$ mpirun -np 2 ./your-model --synch=2  // conservative mode
$ mpirun -np 2 ./your-model --synch=3  // optimistic mode
$ ./your-model --synch=4               // optimistic debug mode (note: not a parallel execution!)
$ mpirun -np 2 ./your-model --synch=5  // realtime optimistic mode
```
