
>[!Introduction]
>As we now from the [[Introduction to computer architecture|first lecture]], we know that memory organization also plays a vital part of the overall architecture.

![[memory-hierarchy-01.png]]

As we see, there are three layers of memory:
- The first one is the secondary memory, with the disk, these are very cheap, easy to produce and can be purchase at a large volume, but it takes forever to process the data (in term of processor's speed).
- The main memory (mostly known about RAM and ROM) - very quick, more expensive than the secondary memory, program is loaded into these memory, depends on the property of the program (temporary or must be persistent) RAM or ROM will involve.
- Lastly we have the cache - which acts as a bridge to narrow the gaps of speed between the processor (CPU) and the main memory. We also have many levels of cache, as shown in the above figure.

Virtual memory and cached are implemented to increase the efficiency of code executing - which helps improving the speed of the system. So how the memory was organized to achieve this? The answer is the memory-access locality. There are two types:
- Temporal locality - if a block of memory was accessed before, there is a high chance that block will be accessed again in the future.
- Spatial locality - if a block of memory was accessed before, there is a high chance that the related blocks (neighbor blocks) will be accessed again in the future.
If a block of code exhibits a very good locality of reference - it is highly concluded that code will be executed rapidly. And on the other hand, if the very same code piece has so-so or poor locality, well, the speed will be slower and slower.


# A. Instructions

The instruction set architecture `ISA` is a critical part of computer - program and the execution of these program as binary code (zero or one in the electronics board). We consider the RISC-V R-format (from left to right).

The format instruction is quite simple
```plaintext
Bit 31                                         Bit 0
|   funct 7 | rs2   |    rs1     | funct3 |     rd     | opcode |
7 bits       5 bits    5 bits      3 bits   5 bits       7 bits
```

So we can literally translate the instruction (in human-readable language) into machine code (binary or hex depends on the requirements).

Fileds:
- Opcode = operation code (MOVE, ADD, SUB,...)
- rd = destination register - the destination of the operation (register, address of the destination)
- funct3: 3-bit function code (optional, not all functions have this field)
- rs1: the first source register number
- rs2: the second source register number (applies for operation with 2 elements)
- funct7: 7-bit function code (additional code)

For example:
`add x9, x20, x21`
We will have the code as:
0000000 - add (funct7)
10101 - rs2 (x21)
10100 - rs1 (x20)
000 - add (funct03)
01001 - rd (x9)
0110011 - add (opcode of add)

-> 0000 0001 0101 1010 0000 0100 1011 0011 is the fully form of binary code for `add x9, x20, x21` - shorten in hex form is: 015A04B3

With I-format (immediate format), we have a little bit different in the anatomy of the code
```plaintext
|     immediate     |    rs1     |    funct3    |    rd     |   opcode    |
  12 bits             5 bits         3 bits         5 bits      7 bits
```

In this form, the immediate acts as an offset added to the based address, 2s-complement, sign extended.

>[!Note]
>Design principle: good design demands good compromises.
>- Different formats complicate decoding but allows 32-bit instruction uniformly.
>- Keep the formats as simple as possible.


![[memory-hierarchy-02.png]]
# B. Pipeline

The pipeline - like the water flows in the pipe, meaning that, for a given execution, we know its have many phases: fetch instruction, decode instruction, compute the result, data access,... and, to maximize the performance of system, these phases can be done parallel, as long as they do not require the accessibility to a same component (data, cache, location, register,...).

![[pipeline-in-action-01.png]]
As shown in the figure, with 3 sequentially commands, each cycle (each command) will take 800 microsecond to complete because the next command must wait for its predecessor. But with the pipeline, because the instruction fetch can be done independently (it does not require any additional resource during this phase), each cycle only takes 200 microseconds (the time for instruction fetch phase).

To understand thoroughly about the pipeline, we must know that, for any instruction, these steps will be carried out:
- Instruction fetch: address instruction memory and read next instruction.
- Instruction decode: transform the instruction into binary code, also read the content of the register.
- Instruction execution: do the computation stuff, including load, store memory address, evaluate branch comparison.
- Memory: evaluate the next address of the memory - related to this instruction.
- Write back: store result on the destination or register file.

## Hazards

This is not a good thing in the pipeline - cause it will make every estimations, every expectations can be unrealistic.

There are three main types of hazards:
- Structural hazards - a required resource is busy.
- Data hazards - need to wait the previous instruction to release that data (read/write), quite common with shared resources.
- Control hazards - deciding on control actions, which depends on previous instruction.

### 1. Structural hazards
![[structural-hazards.png]]

In the beginning, both instruction and data use the same memory (same cache, same storage unit,...). So we can see that in the cycle 4th, for the instruction `add t4, s6, s7`, the IF stage (fetching) will be missed, since the shared memory is busy with the memory access from the 1st cycle. That incident has it own name: bubble - since it shifted the execution 1 time unit further, like the bubble try to float to the surface.

The solution - separate the memory of instruction and data.

### 2. Data hazards

>[!Data dependencies]
>Before going to the main concept, let's take a look about the relationship between write and read operations.

Read-after-read: literally the most pleasant one, just a bunch of sequentially read operations, no dependency after all.
Read-after-write: true dependence, since the value we got affected by write operation, we can only see the result after modification.
Write-after-read: anti-dependence, a troublesome relationship, because it will overwrite some data that might still be needed for some read operation (hence, we got the unexpected result).
Write-after-write: two instructions write same architectural register ... a disaster, may lead to race condition.

Mostly, it will be read-after-write, common case with ordered execution. But if out-of-order execution is allowed, these later two can be dominant type of hazards.

To address this challenge, we need to know one more thing: when is a value available. It really depends on which type of producer handling that value.
- ALU operation can release a value at the end of execution phase, the execution step of next cycle will need it. Forwarding can solve this case.
- Load operation can release a value at the end of MEM - also need for execution step of next cycle (you need resource for computation right), cannot be forwarded.
- Branch operation - well, it very hard, but at the end of execution, when result is available, we can evaluate the condition, and will be needed for IF (the earliest), and cannot be forwarded too.

One of the most important note: values can only be forwards in space - linear, cannot move back in time (at least we have not invented a time travel method lately).

![[data-hazard.png]]
As shown in the figure, the `add` operator will write some result into register x19,  which is also a source register in the next command (hence, as we talked before, will be release the earliest at the end of EX step). 
The result, apparently, bubble for many time, at least after EX, the second command can be executed. Since it needs the result of `x19` before computing.

Bypassing - or forwarding is a technique that allows the result to be used when it is computed - skip the waiting time, do not need to write into register - freshly baked. But, the convenient came with a trade-back: interconnection for data flow.

![[forwarding-in-data-hazard.png]]
In the above illustration, for the first case, no optimization, the result `t2` is only available after write-back step, since it is a register. so the bubble will occur at least 3 steps, after fetching, until decoding.

The second case, a little bit better since the result can be fetched during the first half of write-back, hence the second command can grab it right in the decoding (2nd half requires values).

And the final case, forward from EX to the beginning of the other EX.

### 3. Control hazards

This is a very difficult one, because:
- Longer pipeline can not readily determine outcome easily (cause it contains many commands, operations, and relates to many resources).
- Predict the outcome of the branch.

# C. Memory

## Cache

A component used to store the data you use the most - so CPU does not have to fetch it from the secondary (which is extremely slow in comparison to speed of the CPU).

### Missed
- On hit, CPU proceeds normally.
- Miss:
	- Stall the CPU pipeline
	- Fetch block from next level of hierarchy.
	- Instruction cache miss - restart instruction fetch
	- Data cache miss - complete data access
### Anatomy

The cache has 2 important thing: tag and data. Let think that the task is equivalent to the identifier and inside the data, we have index and offset. Index determines which set should be accessed and the offset told us how far the data lies from the beginning of the set.

## Virtual memory

Main memory is used as a cache for secondary memory (the disks) - these will be managed jointly by both the operating system and the CPU.

Programs share main memory
- Each will get a private virtual address space holding the frequently used code and data.
- Protected from other programs.

CPU and OS translate virtual memory to physical address.
- It is called a page for each block.
- Miss page = page fault.

