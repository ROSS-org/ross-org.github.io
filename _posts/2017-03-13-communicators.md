---
layout: post
title:  "MPI Communicators in ROSS"
author: Caitlin Ross
category: ross-dev
---

By default, ROSS will use MPI_COMM_WORLD for all its communications. `tw_comm_set` allows the user to change the communicator used by ROSS, for instance to run ROSS on a subset of the ranks of a larger MPI application. `tw_comm_set` should be called **before** calling `tw_init`. If `tw_comm_set` is used, then the user is responsible for calling `MPI_Finalize` after calling `tw_end`.

**Note to ROSS developers:** the addition of this functionality changes the internal communicator used by ROSS. From now on, `MPI_COMM_ROSS` should be used instead of `MPI_COMM_WORLD`.
