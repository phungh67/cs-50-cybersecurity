
>[!Narrative]
>Currently, all lectures so far are only considering the single core - that is, a system that has only 1 core to fetch and to manage the instructions.

Check the [[Cache, memory hierarchy - cache missed and related problems]] for memory problem, while the [[Dynamic scheduling and Tomasulo algorithm]] and [[Branch prediction - Dynamic and Tomasuto-based]] related to the performance and the efficiency of the instruction decoding/fetching,...

# A. Thread-level parallelism

>[!Process]
>A program that can run independently of other program on a single or a multi-cores system.

>[!Thread]
>Just a piece of a large program that can be run/scheduled on a processor

For example, a product of two matrix can somehow be divided into many parts, that on one thread, it may find the inverse-matrix, while other part will try to multiple some of them with each other. That how we call "multi-threads".

# B. Shared-memory parallel program

A program that can be divided into more than one parts must deal with a shared memory. The key in this scenario is that, how to ensure the coherence and the earlier part of program (that somehow completed first) should not overwrite or somehow make a memory conflict - or the worst scenario is race-condition

It must be ensured that all threads have arrived before anyone is allowed to continue. And inside the critical section, it is guaranteed that sum is updated atomically (one-by-one, respect the order of the program/algorithm).

# C. Mutlithreading

Increase the resource utilization by multiplexing the execution of multiple threads on the same pipeline (both concurrently and parallelism).

There are basically 2 approaches:
- Switch for each cycle: P1, then P2, then P3,... then P1 (periodically maybe).
- Switch on stalling, every time a process is stalled, switched to another that can be run.

## Simultaneous multi-threading

# D. Organization of multiple-processors

Normally, there must be a shared memory so that every processor can get the data and information in a unified way. The challenge is how to provide low latency and high bandwidth for these accessibilities.
The concept of a hallway: memory on one side and other side is the processors. They can broadcast the need, the complement,... But it comes with zero cache - very slow access speed.
So a layer of shared cache is also appearing right on top of memory, but one level below the processors. But this design introduces a problem of cache-accessing is also on the same path as critical path (interconnection between processors, memory,...).
So the private cache model was brought in. But it also added the data redundancy problem: multiple copies of the same data can exist.

# E. Cache coherence

>[!Problem]
>As in the previous section, it came to that, a multi-layers memory (contains both shared cache and private cache) should be used to deal with the problem of the multi-processors/multi-threads architecture.

