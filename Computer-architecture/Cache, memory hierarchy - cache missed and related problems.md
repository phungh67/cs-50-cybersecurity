
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

Classification of Cache Misses

3-C missed model:
- Cold missed - compulsory, missed when cache is just initialized.
- Capacity missed - miss when the cache cannot hold everything the program needs.
- Conflict missed - two blocks map to the same live.

Cache inclusion and exclusion
- Inclusion is a phenomenon that a data block that occupies at every level of caches.
- Exclusion - the opposite of above concept, no duplicate, high cache utilization but replacing a block requires complex logic and handler.