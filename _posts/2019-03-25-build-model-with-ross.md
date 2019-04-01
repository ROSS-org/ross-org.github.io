---
title: "Building a Model with ROSS"
date: March 18, 2019
layout: post
category: setup
---

The first step for any new model is to get some bare-bones code building.
Unfortunately, building a model on top of ROSS is somewhat complicated.
This post will take you through several methods to getting your code running, each of which are demonstrated with the [template model](https://github.com/ROSS-org/template-model).

## Setup 1: Build your model within `ROSS/models`

The most straightforward way to build a model is to leverage the existing infrastructure created by ROSS itself.
This means creating your model within a sub-directory inside of `ROSS/models`.
From here, the ROSS CMake scripts will detect your model and automatically build it with ROSS.
To allow for models to be kept in a version controlled repository separate from ROSS, users should use a sym-link to "place" their model within `ROSS/models` hierarchy.

This is the process which is currently documented on the [Manual ROSS Installation](https://ross-org.github.io/setup/installation.html) page.

## Setup 2: Have ROSS as a git submodule

An easy to include ROSS into you project is via git submodules.
This setup pins your model to a specific ROSS version, which may prevent issues as ROSS development continues.

To add ROSS as a git submodule to an existing git project, simply run:

{% highlight nil %}
git submodule add https://github.com/ROSS-org/ROSS
git commit -m "added ROSS submodule"
{% endhighlight %}

The project setup then looks like this:

{% highlight nil %}
model/
├── CMakeLists.txt
├── ROSS @DOE26e58
└── src
    ├── CMakeLists.txt
    ├── model.c
    ├── ...
{% endhighlight %}

In this CMake project, the `ROSS` and `src` directories exist side-by-side and are built together from one top-level `CMakeLists.txt` project.

This process is currently used by the [NeMo](https://github.com/markplagge/NeMo) project and is documented on [their wiki](https://github.com/markplagge/NeMo/wiki/Installation-Guide).


## Setup 3: Keep ROSS separate

A more traditional project setup would be to keep the ROSS library entirely separate from the model project.
A user would build and install the ROSS library separate from building their model code.
This method works by leveraging CMake's `FIND_PACKAGE` infrastructure to link the projects together.
Unfortunately, it requires a bit more setup than other methods and users must create a [`FindROSS.cmake` script](https://github.com/ROSS-org/template-model/blob/cmake/cmake/FindROSS.cmake).

A functioning example can be seen on the [cmake branch](https://github.com/ROSS-org/template-model/tree/cmake) of the template-model repository.
