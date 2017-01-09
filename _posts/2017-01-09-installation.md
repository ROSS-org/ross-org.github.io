---
layout: post
title:  "ROSS Installation"
author: Caitlin Ross
category: setup
---

## Requirements

ROSS requires MPI.  We typically use [MPICH](http://www.mpich.org/).  


## Build 

ROSS can be installed quickly by following these simple steps:

- Download the source:

```
$ git clone https://github.com/carothersc/ROSS.git
```

- Create a new ross-build directory:

```
$ mkdir ross-build
```

- Change directories to ross-build:

```
$ cd ross-build
```

- Set the following variables:

```
$ export ARCH=x86_64
$ export CC=mpicc
```

- We use [CMake](www.cmake.org) to build ROSS. <br/>
```
$ ccmake ../ROSS
```
  * You'll want to change `CMAKE_INSTALL_PREFIX` to the directory you want ROSS installation files, e.g., `$HOME/ross-build/install`
  * If you're using one of the ROSS models, you'll want to set `ROSS_BUILD_MODELS` to ON.
  * You may also want to set `CMAKE_BUILD_TYPE` to Debug.

- Finally, we can build:

```
$ make
$ make install
```

