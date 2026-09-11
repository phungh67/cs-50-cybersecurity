
# Limitations of Static Scheduling

>[!Notes]
>Normally, instructions are stalled in `ID` (instruction decoded) util they are hazard-free and then are scheduled for execution.
>The compiler strives to limit the number of stalls in ID.
>However, compiler is limited to what it can do (yep, because its main mission is translating from human-readable code into binary or at least hex code).

**Strength**: 
- Simple hardware and potentially higher clock-rate.
- Power or (and) energy advantage.
**Weakness**:
- Static scheduling sees only compile time information (not run-time).
- Cache misses and branch outcomes appear only at run time.
- Memory addresses may not be known statically.

>[!Conclusion]
>Dynamic scheduling separates dispatch from operand readiness, so independent work can move around stalls. Which means, they will be move around (instructions), so the ready can be executed first (but respect the execution order).

# Overview of the dynamic scheduling

>[!Questions]
>Challenges for this approach? Since the operands are separately handled from readiness stage. So it introduces several take-aways.

**Dispatch first**: Decoded instruction enter queue even if the operands are not ready (source-1, source-2,...).
**Wait in queue**: Instructions will wait util operands and executions units are available (ALU,...).
**Issue by readiness**: Execution is carried out by data-flow (not just by program order).

>[!Challenges]
>All register hazards can appear: Read-after-write, write-after-read, write-after-write.
>Memory dependencies maybe unknown until addresses are computed (since they are put into queue).
>Branches and exceptions must still preserve the program model.

# Tomasulo algorithm

>[!Note]
>As described earlier, this algorithm puts instructions into queue (or reserve units), so the pipeline can be busy at all time. Every time that an operand is unable to move on (that is, lacking any resource or lacking a data maybe), it will be put in the queue.


Normally, when an instruction is stalled, the pipeline will also be stalled, then wait for that instruction to be executed again - this creates a massive waste in term of performance. Then, the dynamic scheduling can improve it by allowing out-of-order execution, every instructions that are stalled can be moved into respectively queues (memory, integer, floating), then another one can go to the ID stage, maximize the throughput (instruction per cycle, or cycle per instruction).

Then, all components will be connected by the `CDB` - Central Data Bus, like a highway for all communication purposes (mostly data transfer). Every time a data, a register or any instruction has released its common register or something similar, it will broadcast the message in this highway, then all waited instruction can go and grab necessary sources for their execution.



## Register renaming

All the data dependencies can be solved by register renaming. For example, if an operand is progressing further, it will be stored. Producer instruction's ID is tagged in the operands of the consumer instruction. Since the producer instruction's ID was referred in later, so every time another dependency comes, it just look the table.

### Front-end

An instruction will firstly moved to Fetch Queue FIFO (Fetch-instruction stage), then after decoded, to the issue queue (ready-to-go). An entry is located in the respective issue queue (integer, floating, memory depends on the operand), regardless the readiness (maybe some of the operands are computed, in-used,...). The tag - for the instruction that will produce this register value, and v for valid or not. Example: `T1, -, F1, 1` : the register T1, stores the value of instruction F1 (- means it is still being computed now, 1 is valid). If the F1 somehow finishes, it will broadcast to the `CDB` then the T1 will store the actual value in it.

### Back-end

Instructions wait in the issue queue (reservation units) util all data and structural hazards are solved.

Operands whose values is pending will be propagated over the `CDB` when the dependent instruction finished its execution.

An instruction can be issued for execution if:
- All data is available (ready).
- No structural hazard of the FU unit or the `CDB` after execution (for short: in available, out must available too).

If an instruction finishes its execution, it will also broadcast a signal <Val,T> tuple, that:
- Indicates queue entries that contains operands with tag `T` should be ready for execution.
- Register file entries whose tag match `T` should grab the value, update in the file (front-end) then change the valid bit into 0 - ready.


