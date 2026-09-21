Prior to this lesson, we assumed  that every memory instruction only took 1 cycle to complete. In a reality perspective, it is unrealistic, hence impossible to achieve. So we have to take the memory hierarchy and memory accessing penalty in account.

# I. Cache hierarchy and performance aspect.

>[!Notes]
>Since we have the memory wall, which is the speed of developing new memory is extremely slow in comparison to the speed of releasing new CPU (and even not, we still have the multi-cores era - many core. a.k.a the processing units in one chip).

We also have the hierarchy of memory - multi layers, multi-levels of cache to deal/mitigate the memory wall.

The processor will interact directly with the Level - 1 cache, if missed, level 2 cache will be involved. Otherwise, it fetches the data from the memory (maybe main or secondary), which takes forever in term of speed - comparison to speed of processor and level 1 cache.

We also have a general metric to measure the effectiveness of memory accessing called Average Memory Access Time:
$$
AMAT = HR_l * T_l + MR_l * MP_l
$$
In which:
- $HR_l$ is the hit-rate, along with $T_l$ is the accessing time of the memory on the same level
- $MR_i$ and $MP_i$ are the miss-rate and the missed penalty on the same level

But we need to assure that, when a cache-missed happened, the miss penalty and the miss-access-time must be fetched from the next level. Because, when a level 1 missed, the processor (or whatever components in charge of data fetching) should look for data on the next level (e.g level 2).

Impact on the execution time:
We have the ${CPI}_0$ is the idea time of execution - without any cache missed (hit all time). On the other hand we have ${CPI}_1$ is the Cycle-per-instruction for hit cache and ${CPI}_{miss}$ is the CPI for penalty.

At the end, we have
$$
CPI = {CPI}_0 + {CPI}_{hit} = {CPI}_{miss}

| {CPI}_{miss} = MPI * MP
$$

# II. Cache miss - classification and solutions

>[!Note]
>Not every misses are the same. There are misses that must be carried out, hence unavoidable, but there are many misses that can be addressed - improve the performance of the system, and also increase the cache/memory utilization (we want the memory to work as much as possbile).

- Compulsory misses - happens on first reference of any blocks. Even with an infinite cache, there is always the first miss, since there are no data were populated, the cache would return a miss.
- Capacity misses - a cache cannot hold every necessary data of the program, hence, this kind of miss says that, miss happens because of the data replacement.
- Conflict misses - two blocks map to the same line in a direct-mapping of set-associative cache.

>[!Counter]
>There are three properties of a cache: overall size, block size and associativity.

Each property bring a tool to address a challenge (but not all), so a good cache should be a combination of several values, and must maintain the harmony between these arguments.

- A larger cache can mostly tackle the capacity misses (since it can hold many values) but comes with a higher price. It also can address somewhat conflict misses (but not all, and again, need test to fully claim this statement).
- A larger block size may help to reduce the compulsory misses, also can tackle the capacity and conflict misses, since larger block means larger range/fleet of data can be loaded.
- Higher associativity mostly aims to resolve the conflict misses.

# III. Inclusion and Exclusion
## 1. Inclusion

Which literally means a block of data can be duplicated in all cache levels. When a block was missed, it would be brought to all level below the missed level. Example: Block A was requested by a level 1 cache, but missed, so it would be brought to every level below level 1.

It comes with a consequence, when a block was replaced, it also must be removed from every level higher. Block A was replaced in level 2, so any copies of A from level 1 should be removed too.

With the inclusion, the effective capacity of the cache = size of the largest level.

## 2. Exclusion

Exact one copy of a block. If  a block will be replaced, it would be allocated in the next level. Effective capacity is sum of all levels. Good for maximizing the utilization but the replacement is much more complex.

# III. Non-blocking cache

Normally, a cache missed also resulted in blocking - next instruction/data fetching must wait until the missed penalty was resolved. For example, when the MEM execution requested a piece of data, and it was missed - the pipeline must wait for the time that the cache fetched that data from memory (100 nanoseconds maybe).

To successfully achieve a non-blocking cache, there must be some additional mechanism.
The MSHR - Miss Status Handling Register keeps:
- Address of the pending miss.
- Destination of the block in the cache (destination block).
- Destination register.

A new entry in the MSHR will be allocated if:
- Primary miss - the first miss to a block - allocates a new one.
- Secondary miss - later access to a block whose miss is already placed, will attach to the existing MSHR entry.

>[!Question]
>Assume that the block size is four words. Determine the number of Primary and Secondary Miss if the loop is executed 16 iterations. Assume that the number of the MSHR suffices.

```Assembly
LOOP: LW   R1,0(R2)
	  ADDI R2, R2, #4
	  BNEZ R2, R4, LOOP
```

The load command (`LW`) will miss at first. But the cache accessing must go through the entire block, block take 4 words, each cycle moves only 1 step

=> To move to another block: takes 4 cycles for every block. The first miss = first access, then 3 sequentially misses are secondary.

In total: 4 primary misses and 12 secondary misses.

# IV. Cache prefetching

Simple: how to prefetch (or maybe pre-warm?) the cache, so the data will be fetched before any instruction talked to the cache "Hey do you have this data?".

The strides - time of consecutive access, it will trigger a stride detection, then the data will be pre-fetched into the cache.

>[!Question]
>Assume the following:
>Access sequence is 100, 110, 120,... 160.
>Block size is 16 bytes, stride prefetcher brings block in zero time (no additional time for this type of operation)
>How many misses?

16 bytes => there are at least 4 blocks, since 4x16=54 + 110 = 154
So the first block will be 100 - 116, 117 - 133, 133 - 149, 150 - 160

Three accesses = detect => 100, 110 and 120 should be enough to trigger. 120 lies on the second block. So at least 2 missed block. Last 2 blocks will be perfectly fetched.

