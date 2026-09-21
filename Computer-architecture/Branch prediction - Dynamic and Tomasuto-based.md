# Dynamic Branch Prediction

>[!Speculative]
>Which means, any outcome is just a theory, thus results can be varies depends on later execution.

Situation:
- Tomasulo pipeline: instructions after each branch are stalled in the dispatch stage, util the branch outcome is out on the `CDB` (only applies for instructions after a branch, not the normal ones).
- So the parallel level is quite low, since we still have a bunch of instructions waiting for execution.
- Assuming that, static prediction was taken in, instructions were fetched. If the prediction was correct, nothing happened, instructions could be dispatched immediately, but if not, the IF queue must be flushed, restart fetching from the fetch point, costed much more parallelism than normal.

>[!Non-speculative execution]
>Static, a tree of possible results

- All-path execution, hardware intensive, difficult to track, especially with complex program (due to many branches).
- Unwanted exceptions come from wrong-path execution, which should be invisible to save resource.
- Waste of resource - of course, instead of following to only 1 correct path, we have to trace all branches, for the whole possibility tree.
>[!Speculative execution]
>Well, the name says it all: based on "prediction".

- May be more efficient than non-speculative.
- Must be optimized, if not, every miscalculation, misspeculation will cost much more.

## Branch Prediction Buffer

A unit to store the prediction information: what is the identity of that branch instruction, prediction status (taken or not taken) <- which can be changed to reflect if the prediction was correct or not.

### 1-bit branch prediction

Only one bit for an entry. For example, ID = 1, instruction = if, bit = 0 (initial). Prediction: if should outcome true, if correct, status was not changed, if not, the direction of the bit would be changed.

The truth table:

|     | U   | T   |
| --- | --- | --- |
| 0   | 0   | 1   |
| 1   | 0   | 1   |
For example, for a branch prediction, Taken = 1 and Un Taken = 0. Giving the history
01111101011110
Calculate the accuracy of the 1-bit prediction on the branch, assume that initial state is 0?
We have the truth table as above, and from the history + init =0, we have

Predict: 00111110101111
History: 01111101011110

The bit is changed every-time the predict mismatches with the reality.

Correct = same result between predict and history => 8 times
Total of 14 times 
=> Accuracy = 8/14 = 57%



### 2-bit branch prediction

Same as 1-bit, but the additional bit can show about the historical data and the confidence of the prediction.

00 -> 01 -> 10 -> 11: increase 1 for every correct prediction.
Every wrong prediction costs 1,  2 mispredictions in a row are required to change the direction

Example, for the loop `for(i = 0; i < 100; i++)` how many times the 2-bit prediction will miss?

At first, initial state is 00 -> predict is not taken, but the loop was executed, then the value should be 01. But because the MSB is still 0 -> predict is still not take, but because the loop, it taken again, so the bit is 10. That were two misses.

After that, since the MSB is 1, the prediction must be taken. Util the last loop, 11 will be decreased to 10 -> 1 miss

So total 3 misses

## Two levels branch prediction 

>[!Branch dependencies]
>Should keep track the previous branch in correlation with current branch.

### (M,N) Branch Prediction Buffer

Instead of a traditional one (with only 1 or 2 bits) new buffer has its own capacity. For example:
- If the program branch counter has P bits.
- And the branch history has M bits.
The value of the buffer (size) will be $N*2^M*2^P$ with N is the size of the prediction bit.

Since for each bit in the program branch counter and branch history, there are 2 states (that why we have the power of 2 here).

###  Two-level branch predictors

There are 2 levels: Global and Private
- Use for global branching.
- Or private branching (after some specific global branches).

>[!Branch Target Buffer]
>Aim to eliminate the branch penalty. Cache for all branch target addresses.

- Accessed in Instruction-Fetch in parallel with instructions and Buffer entry.
- Relies on the fact that the target's address is never changed (most likely).
- Organized as a direct-mapped cache, thus it does not alias branch addresses.

If there is a miss on the `BTB`, must wait for the branch instruction (similar to the cache missed).

