---
layout: post
title:  "RIO Version 2 Release"
author: Elsa Gonsiorowski
category: rio
---

The latest version of the RIO API for ROSS checkpoints has just been released!
This version has many core features, including event capturing and allowing multiple partitions per MPI rank.
Additionally, all of the RIO documentation has been ported to the ROSS blog.

## Overview

RIO is an API for creating restart checkpoints of a ROSS simulation.
These checkpoints are created after the completion of a ROSS simulation and cannot be created *during* simulation execution.
Thus, RIO is a system for checkpoint-restart operations.

The major achievement in this release is accompanying documentation.
Part of this documentation includes a full implementation of the current API within the [PHOLD-IO model](https://github.com/gonsie/pholdio).
Integrating RIO into an existing ROSS model should be straightforward and simple to understand.

## Future Work

Currently, RIO cannot be used with optimistic simulations.
This is a severe limitation and an optimistic algorithm is first priority in version 3.
