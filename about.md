---
layout: page
title: About
---

ROSS is an acronym for Rensselaer's Optimistic Simulation System.
It is a parallel discrete-event simulator that executes on multiprocessor systems and/or supercomputers.
ROSS is geared for running large-scale simulation models (millions of object models).
The synchronization mechanism is based on Time Warp.
Time Warp is an optimistic synchronization mechanism develop by Jefferson and Sowizral [5, 6] used in the parallelization of discrete-event simulation.
The distributed simulator consists of a collection of logical processes or LPs, each modeling a distinct component of the system being modeled, e.g., a server in a queuing network.
LPs communicate by exchanging timestamped event messages, e.g., denoting the arrival of a new job at that server.

The Time Warp mechanism uses a detection-and-recovery protocol to synchronize the computation.
Any time an LP determines that it has processed events out of timestamp order, it "rolls back" those events, and re-executes them.
For a detailed discussion of Time Warp as well as other parallel simulation protocols we refer the reader to [4].

ROSS was modeled after a Time Warp simulator called GTW or Georgia Tech Time Warp[3].
ROSS helped to demonstrate that Time Warp simulators can be run efficiently both in terms of speed and memory usage relative to a high-performance sequential simulator.

To achieve high parallel performance, ROSS uses a technique call Reverse Computation.
Here, the roll back mechanism in the optimistic simulator is realized not by classic state-saving, but by literally allowing to the greatest possible extent events to be reverse.
Thus, as models are developed for parallel execution, both the forward and reverse execution code must be written.
Currently, both are done by hand.
We are investigating automatic methods that are able to generate the reverse execution code using only the forward execution code as input.
For more information on ROSS and Reverse Computation we refer the interested reader to [1, 2].
Both of these text are provided as additional reading in the ROSS distribution.

**References**

1. C. D. Carothers, K. Perumalla and R. M. Fujimoto. Efficient Parallel Simulation Using Reverse Computation. *ACM Transactions on Modeling and Computer Simulation*, vol 9. no 3. Jul. 1999.
2. C. D. Carothers, D. Bauer and S. Pearce. High-Performance, Low Memory, Modular Time Warp System. In *Proceedings of the 14th Workshop of Parallel on Distributed Simulation* (PADS '00), pages 53&ndash;60, May 2000.
3. S. Das, R. Fujimoto, K. Panesar, D. Allison, and M. Hybinette. GTW: A Time Warp System for Shared Memory Multiprocessors. In *Proceedings of the 1994 Winter Simulation Conference* (WSC '94),  pp. 1332&ndash;1339, Dec. 1994.
4. R. M. Fujimoto. Parallel Discrete-Event Simulation. *Communications of the ACM*, vol. 33 no. 10 pp. 30&ndash;53, Oct. 1990.
5. D. R. Jefferson and H. Sowizral. Fast Concurrent Simulation Using the Time Warp Mechanism, Part I: Local Control. Technical Report N-1906-AF, RAND Corporation, Dec. 1982.
6. D. R. Jefferson. Virtual Time. *ACM Transactions on Programming Languages and Systems*, vol. 7 no. 3 pp. 404&ndash;425, Jul. 1985.

# History of the Code

ROSS's history starts with a one-week re-implementation of [Georgia Tech Time Warp (GTW)](http://www.cc.gatech.edu/computing/pads/tech-parallel-gtw.html) by Shawn Pearce and Dave Bauer in 1999.
After 10 years of in-house development, version 5.0 of [Rensselaer's Optimistic Simulation System](http://sourceforge.net/projects/pdes/) went live at SourceForge.net.
Thus the official version history began!

Through the years ROSS has migrated from CVS, to SVN, to Git and GitHub.com.
The code was maintained by Chris Carothers and his graduate students at RPI ([publications](http://cs.rpi.edu//~chrisc/#publications)).
Over the years, several features (including a shared-memory version) were implemented within ROSS.
Some of these features have since been optimized out, leaving behind cruft.

In early 2015 a sleeker version of ROSS was released.
Developed as Simplified ROSS ([gonsie/SR](http://github.com/gonsie/SR)), this version removed many files, functions, and variables that had become deprecated over time.
