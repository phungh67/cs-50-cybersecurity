
# Static Scheduled Pipeline
![[Quiz-hazards-in-a-timing-table.png]]

As we see, the hazard introduced a new one: out-of-order and out-of-completion for statically instruction/pipeline. Since the FP - stands for floating point execution. For any computation task, the integer and the floating are 2 separated parts, hence leading to a case that some "younger" (child) execution can finish before the main one. Apparently, this introduces more chance of WAW - write after write (overwrite result) or WAR - write after read (missing correct result).

In this quiz, the relation is RAW - read after write. In the cycle 7 and 8 (precisely, the earliest is after cycle 7, beginning of cycle 8), F1 can be read. And then, the read of I2 can be done in FP2 (the most correctly time to have the true result). So, there should be a forwarding from I1 to I2.

Similarly, a forwarding is needed between I3 and I4.

So the correct statements are C and D.

![[extended-pipeline.png]]
The extended pipeline for operation. Arithmetic will come with two separated parts: for floating and for decimal, so it will be extended into FP rather than only EX.

The execution will wait at ID util it is ready to execute (will sufficient resource, data,...)

![[quiz-structural-hazard.png]]

In this case, we clearly have a structural hazard.

For every instructions that involved computing (arithmetic), they will be extended with the floating point steps. On the other hand, the statically command like `LS` - load only consumes a shorter pipeline.

As we see in the cycle 8 (8th cycle) both `I1` and `I5` require the same memory (the memory caching) - structural hazards due to access to a same resource.

Also, 9th cycle introduces the very same problem: both instructions entered the same structural - write back.

So, we faced 2 problems:
- Stall - longer latency - larger execution time - worse performance.
- More resource (memory,...) - higher consumption, higher area in the board.

Moreover, longer execution latency and out-of-order completion change make the dependencies (a sign of data hazards) more visible.

![[precise-exception.png]]

Another concept is **precises exception**, in this case there are two instructions, even they use different register, they are still related to each other.

Note that instruction 2 has shorter execution time in comparison with instruction 1, that leads to many problem.

In a case that instruction 1 encountered some fault, error,... the operation system may init the error recovery, maybe repeat the execution of instruction 1, maybe discard it,... and because these 2 instructions are related to each other (from a same piece of code) - the instruction 2 maybe re-executed - waste time, cycle, power.

-> Solution: force the in-order execution, for example, with the statically one (integer), force a feedback after memory, then merge that result with floating point, and these 2 have a common write back at the end. It limits 2 things:
- Freedom in the pipeline.
- Extra cost about control units, data storage,...


>[!Superpipeline]
>Split stages to increase clock rate, shorter cycle time.

But, it leads to longer cycle penalties, since more cycles - more latencies count.
And also, it could not resolve the dependencies problem, faster clock is purely about speed, not the hazards (still need the same memory, still need to wait for data). Superpipeline - split work into smaller parts so parts can be done parallel or at least, no super long idle time, it does not eliminate the logical dependencies between stages.

>[!Superscalar CPU]
>Allow more than one instruction per cycle.

But it must come with rule: pairing one integer/branching/memory instruction with one floating point instruction (so that they are unlikely to have conflict with each other). In addition, these 2 instruction must be independent with each other - no hazards.

Lastly, 2 is good, but 3 is very complex to design, implement.


# Static Instruction Scheduling

## Static Branch Prediction

### Hardwired prediction
- Always predict untaken in the simple model and execute in the EX stage (the execution stage).
- No compiler-assist
- Loop-crossing branches are often mispredicted
- Decision is made at design time using workloads

### Compiler-time prediction
- Contain a bit-hint for each branch.
- Compiler uses profiling or static information and sets the bit accordingly.
- More flexible than global rule.

Note - they only use the information before execution, cannot use run-time behavior.

### Static instruction scheduling

The compiler chooses an instruction that expose more useful work to the pipeline - maximize the `IPC` - instruction per cycle.

It leads to a concept called **Local scheduling** - as said before, the compiler chooses instruction (move instruction) in a controlled manner to improve the performance metrics (instruction per cycle, cycle per instruction) (especially in loop).

Need to be careful with any instruction that touches the address. For example, instruction A, after B, need 1 address jump, move A further away -> need to match the address again in instruction A.

Take this example:
```Assembly
Loop: L.S   F0, 0(R1)   (1)    <--- Latency in ID stage = 1 + #stall_cycle
      L.S   F1, 0(R2)   (1)
      ADD.S F2, F1, F0  (2)
      S.S   F2, 0(R1)   (5)
      ADDI  R1, R1, #4  (1)
      ADDI  R2, R2, #4  (1)
      SUBI  R3, R3, #1  (1)
      BNEZ  R3, Loop    (3)
```

So, it will take 15 cycles to complete 1 loop.
Latency: 
- For every load command, only take 1 since it does not need to wait, so no stall.
- For add command, since the first one depends on the load command, so +1
- Store command needs F2 - wait for add, then, 2 for load, 2 for add and 1, then 5
- The BNEZ takes extra penalty cycles.

We see that the `SUBI R3, R3, #1` only needs `R3` - which is independent - so we can move the command to position #3, so that once `ADD.S F2, F1, F0` was taken in action, it does not suffer cycle stall.

But since the `ADD.S` was moved down, right after the `SUBI`, the `S.S` should be modified a little

```Assembly
Loop: L.S   F0, 0(R1)   (1)    <--- Latency in ID stage = 1 + #stall_cycle
      L.S   F1, 0(R2)   (1)
      SUBI  R3, R3, #1  (1)
      ADD.S F2, F1, F0  (1)
      ADDI  R1, R1, #4  (1)
      ADDI  R2, R2, #4  (1)
      S.S   F2,-4(R1)   (3)
      BNEZ  R3, Loop    (3)
```

As we see, the `ADD.S` only needs to wait the `SUB.I` - the stall cycle decreased to 1.
Moreover, the `S.S` suffers 3 cycles at most (distance from `ADD.S` - with `F2` and two additional `ADDI` commands). But, since `R1` was just increased 4 by the command `ADDI`, if storing `F2` to `R1` again, we need to respect the original command `S.S F2, 0(R1)` - so that is the reason why there is `-4`.

Therefore, only 12 cycles are needed. The speedup is 15/12.

### Loop unrolling

(like expand the loop into a larger body)
```Assembly
Loop: L.S   F0, 0(R1)   (1)    <--- Latency in ID stage = 1 + #stall_cycle
      L.S   F1, 0(R2)   (1)
      ADD.S F2, F1, F0  (2)
      S.S   F2, 0(R1)   (5)
      ADDI  R1, R1, #4  (1)
      ADDI  R2, R2, #4  (1)
      SUBI  R3, R3, #1  (1)
      BNEZ  R3, Loop    (3)
```

We have the original piece of code. Then expand to 2 loops:
```Assembly
Loop: L.S   F0, 0(R1)   (1)   
      L.S   F1, 0(R2)   (1)
      ADD.S F2, F1, F0  (2)
      S.S   F2, 0(R1)   (5)
      ADDI  R1, R1, #4  (1)
      ADDI  R2, R2, #4  (1)
      SUBI  R3, R3, #1  (1)
      BNEZ  R3, Loop    (3)
      L.S   F3, 4(R1)   (1)   <---- change because must avoid WAW
      L.S   F4, 4(R2)   (1)   <---- same reason
      ADD.S F5, F3, F4  (2)   <---- same too
      S.S   F5, 4(R1)   (5)   <---- 4 cause by next iteration
      ADDI  R1, R1, #8  (1)   <---- same reason, cause we check 2nd iteration
      ADDI  R2, R2, #8  (1)
      SUBI  R3, R3, #2  (1)
      BNEZ  R3, Loop    (3)
```

Since we unwrapped loop by combining 2 loops into 1, reorganize should be carried out

```Assembly
Loop: L.S   F0, 0(R1)   (1)   
      L.S   F1, 0(R2)   (1)
      L.S   F3, 4(R1)   (1)   <---- change because must avoid WAW
      L.S   F4, 4(R2)   (1)   <---- same reason
      ADD.S F2, F1, F0  (1)
      ADD.S F5, F3, F4  (1)   <---- same too
      ADDI  R1, R1, #8  (1)
      ADDI  R2, R2, #8  (1)
      SUBI  R3, R3, #2  (1)
      S.S   F2,-8(R1)   (1)
      S.S   F5,-4(R1)   (1)
      BNEZ  R3, Loop    (3)
```

>[!Note]
>Unrolling works best when iterations are independent and code-size/register costs are acceptable

