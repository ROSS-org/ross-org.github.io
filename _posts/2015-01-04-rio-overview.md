---
layout: post
author: Elsa Gonsiorowski
title: "Overview"
category: rio
---

> E. Gonsiorowski. "Enabling Extreme-Scale Circuit Modeling Using Massively Parallel Discrete-Event Simulation,” Ph.D. dissertation, CS, RPI, Troy, NY, 2016.

Parallel discrete-event simulation is a common application for HPC resources. 
Large simulations can often benefit from an increase in compute power, sometimes experiencing super-linear speedup. 
Thus, running the same simulation on multiple, different parallel configurations is valuable. 
Long running simulations may run into time limits imposed by the HPC resource provider. 
Here, the ability to save an entire simulation state to disk and then restart the same simulation from the saved checkpoint is essential.

There are several, existing APIs for performing file I/O within massively parallel applications, including those defined by the popular Message Passing Interface (MPI) standard. 
These API are flexible and built to work with many types of parallel applications.
While this flexibility benefits developers, application users often need a much smaller set of capabilities. 
Indeed, PDES users (i.e., the model developers) should not have to understand the inner workings of their chosen PDES framework.

This section presents ROSS I/O, called RIO, an API which facilitates PDES restart-checkpoints by providing a user-level interface to parallel I/O functionality, RIO is able to take advantage of the many “known” quantities of PDES, namely that the entire system is defined from LPs and events. 
RIO is able to create a compact, yet flexible file format for checkpoints, where metadata is kept separate from actual simulation system state. 
This interface also encapsulates the logic of mapping the data within the checkpoint to MPI processes in their parallel arrangement.

RIO is completely integrated into the ROSS API and, like other ROSS features, is implemented by the model developers through function callbacks. 
Its main purpose is to allow model developers to create checkpoints at the conclusion of their serial or conservative simulations. 
These checkpoints can then be used to restart and continue the simulation. 
One key feature is RIO’s dynamic capability which allows for a change in parallel configuration across executions of the same simulation.

