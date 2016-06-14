---
layout: post
title:  Code Organization and Files
category: model-dev
---

This post briefly describes the best practices for organizing the code for a ROSS model.
These guidelines differ from what is typically recommended for large object oriented projects, so programmers familiar with C++ or Java should take note.

## Main Header File: model-name.h

Unlike many OO programs, ROSS models typically have one, main header file.
This file should do the following:

1. `#include <ross.h>`
2. Define enumerations and typedefs.
3. Define the message struct.
   If many message structs are needed, a `union` should be used to create a single message type for the simulation.
4. Define the LP state structs.
5. `extern` any global and command line variables.

## LP
