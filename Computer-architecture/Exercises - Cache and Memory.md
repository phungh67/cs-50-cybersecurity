Theories and principles [[Cache, memory hierarchy - cache missed and related problems]]

Review the 3-C model: Cold Misses, Capacity Misses and Conflict Misses

The cold miss is the miss that must happen (due to that entry has not been in the cache before)
# Problem 1
First-level instruction cache are often directed-mapped, cause they are better at handling loops than set-associative caches.
We have 4 lines for caching and total of 6 addresses that can be accessed (0 to 5), repeated 10 times. We have to calculate the misses (all types) into categories.
Considered that direct-mapped, FA with OPT replacement, FA with LRU, FIFO, LIFO and 2-way Set-Associative cache with LRU.

Direct-mapped: line mapped to block (block % number of lines)

| Iteration | Address | Line 0 | Line 1 | Line 2 | Line 3 | Comment                                      |
| --------- | ------- | ------ | ------ | ------ | ------ | -------------------------------------------- |
| 1         | 0       | 0      |        |        |        | Cold miss                                    |
|           | 1       | 0      | 1      |        |        | Cold miss                                    |
|           | 2       | 0      | 1      | 2      |        | Cold miss                                    |
|           | 3       | 0      | 1      | 2      | 3      | Cold miss                                    |
|           | 4       | 4      | 1      | 2      | 3      | Cold miss, Direct mapped, 4 = 4 % 4 = 0 line |
|           | 5       | 4      | 5      | 2      | 3      | Cold miss                                    |
| 2         | 0       | 0      | 5      | 2      | 3      | Miss (conflict)                              |
|           | 1       | 0      | 1      | 2      | 3      | Miss (conflict)                              |
|           | 2       | 0      | 1      | 2      | 3      | Hit                                          |
|           | 3       | 0      | 1      | 2      | 3      | Hit                                          |
|           | 4       | 4      | 1      | 2      | 3      | Miss                                         |
|           | 5       | 4      | 5      | 2      | 3      | Miss                                         |
So for each iteration (after the first) will have 4 misses
Total misses: 6 (first) + 9 * 4 = 42 misses

Optimal 
Evict the block that won't be used in the longest time in the future
Pattern: (0,1,2,3,4,5) -> the 3 would be evicted

| Iteration | Address | Line 0 | Line 1 | Line 2 | Line 3 | Comment |
| --------- | ------- | ------ | ------ | ------ | ------ | ------- |
| 1         | 0       | 0      |        |        |        | Cold    |
|           | 1       | 0      | 1      |        |        | Cold    |
|           | 2       | 0      | 1      | 2      |        | Cold    |
|           | 3       | 0      | 1      | 2      | 3      | Cold    |
|           | 4       | 0      | 1      | 2      | 4      | Cold    |
|           | 5       | 0      | 1      | 2      | 5      | Cold    |
| 2         | 0       | 0      | 1      | 2      | 5      | Hit     |
|           | 1       | 0      | 1      | 2      | 5      | Hit     |
|           | 2       | 0      | 1      | 2      | 5      | Hit     |
|           | 3       | 0      | 1      | 3      | 5      | Miss    |
|           | 4       | 0      | 1      | 4      | 5      | Miss    |
|           | 5       | 0      | 1      | 4      | 5      | Hit     |
| 3         | 0       | 0      | 1      | 4      | 5      | Hit     |
|           | 1       | 0      | 1      | 4      | 5      | Hit     |
|           | 2       | 0      | 2      | 4      | 5      | Miss    |
|           | 3       | 0      | 3      | 4      | 5      | Miss    |
|           | 4       | 0      | 3      | 4      | 5      | Hit     |
|           | 5       | 0      | 3      | 4      | 5      | Hit     |
| 4         | 0       | 0      | 3      | 4      | 5      | Hit     |
|           | 1       | 1      | 3      | 4      | 5      | Miss    |
|           | 2       | 2      | 3      | 4      | 5      | Miss    |
|           | 3       | 2      | 3      | 4      | 5      | Hit     |
|           | 4       | 2      | 3      | 4      | 5      | Hit     |
|           | 5       | 2      | 3      | 4      | 5      | Hit     |
