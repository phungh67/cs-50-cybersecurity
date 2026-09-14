
# Dynamic branch prediction

Normally, the instruction after branch are stalled in the dispatch stage, util the branch outcome is out of the `CDB`.

Apparently, the branch is statically predicted. If it was correct, nothing special, but if not, we will lose even more.

>[!Non-speculative execution]
>Execution is a tree of possibilities. Not executing all paths after branch because of wasting resource and it will be a very intensive hardware-requirement design.

>[!Speculative execution]
>Only execute the path that is the most likely happened.

## Branch prediction buffer

Is a place that we place some "predicted" instructions in it. So if we did some wrong predict, it would affect too much to the pipeline. Index with the bits of the branch address. Predict is taken or not, and if wrong, roll back and update the register (hold the prediction value again).

### 1-bit branch predictor

- Each Branch Prediction Buffer `BPB` only has 1 bit.
- Update the bit after the branch is resolved.
- Predict the same outcome as last time (if meet same situation)
For example, calculate the branch prediction accuracy for 1-bit predictor, initial state is 0, and history is 01111101011110.

We know that 0 is un-taken and 1 is taken. 0 taken becomes 1, and 1 un-taken becomes 0.

### 2-bit branch predictor

- Additional bit to show the confidence. Begins from 00 to 11, plus 1 for correctness, and minus 0 for the opposite.

For example, for a simple loop `for(i=0; i<100; i++)`, how many times a 2-bit predictor mispredict it? 3 times: initial, loop entrance and loop exit

## Branch dependencies

To improve the branch prediction, must take account the result of the previous branches in the progress.

A single branch can behave differently on different context.
Correlation is: if the 2 first one is not taken, likely that the third one is not taken too.

Use N-bit predictor with:
- P bits from the branch program counter.
- M bits from the branch history register.
- So the `BPB` size is $N*2^M*2^P$ 

## Branch Target Buffer

Store the data about the target address (next address, the address that should be taken after the IF command).
The BTB will cache all branches's target

Example: The following program
```C++
for(i = 0; i < 100; i++){
	if (i % 2 ++ 0){
		then a = 1;
	} else {
		a = 0;
	}
}
```
We know that 1 is taken and 0 is not taken. While if branch is taken for even i. And the loop-crossing branch is taken for first 3 i (0,1,2) and the oldest history is show on the left. The value of the global branch history register.

Global = taken all conditions, not just current branch.

There are 2 branches: For loop and If.

- For: taken for 3 first value => we have 111
- If: only taken for even => 1 (0) - 0 (1) and 1 (2)
- So we have 110111 (oldest to newest)

## Hardware supported speculation
