
>[!  Current situation]
>First of all:
>- Many CERT security incidents today are still buffer overflows.
>- C and C++ in generally do not perform array bound checks.
>- Possible to write past the end of an array and plant malicious code (payload) or highjack control (by overwriting the returning address).

To understand better about this attack at a deep-under-the-hood level, let see the structure of a `Call Stack` (the memory in which a program resides during its execution).
```text
		   high addresses 
fp ------> +ooooooooooooooo+ |
           +ooooooooooooooo+ |
           +ooooooooooooooo+ |
sp ------> +---------------+ |
           +---------------+ |
           +---------------+ |
           +---------------+ |
           +---------------+ |
           +---------------+ |
           +---------------+ v
           low addresses 
```

We have 2 pointers, the first one is `fp` - frame pointer, points to the start of  the stack, this pointer will move depends on how many time another functions were called during the program's execution.

The second one is the `sp` - stack pointer, point to the currently executed address.

```Go
// begin to call a function
push arg1;...; push argN // push the necessary arguments into the stack
push return_address; // the address just next to the function once it completed

// within the callee (the called function)
push fp; // save the old frame pointer to return
fp := sp; // move the frame to current stack
sp := sp + sizeof(local_vars); // move to spend enough spaces for args
// complete setup and execution
sp := fp; // move stack pointer back to frame pointer, delete all func vars.
fp := pop(); // recover old frame
pc := pop(); //
```

Currently vulnerable functions: `strcat`, `strcpy`, `scanf`, `sscanf`, `gets`, `read`,... since all of them do not provide any boundary checking.

# 1. Shellcode

In the `Linux`, the main any maybe most "programmer-alike" way to use it is through a `Shell` - an interface allow users to freely navigate themselves in the Operating System. But first of all, there are several things to keep in mind:
- Shell code spawns a shell under the `uid` of the caller process, which means, if the `uid` was elevated to root, this will give you a `root` shell.
- How to make sure buffer address overwrites return address?, well we have something like this in the address space: `[shellcode][ADDR][ADDR][ADDR]...`
- And how to make sure the return pointer will point only to shell code? Implement the No-op sled, leaving a vast space and the hacker can jump fast: `[NOP][NOP][NOP]...[shellcode]`.
The `gcc` and `gdb` can be used to extract assembly and hex representations. Also, the `NOP` on the x86 system has the machine code of `0x90`. And that leads to the question "How do you guess the ADDR to put the payload":
- The first stack frame should be found first, and this is deterministic in some Linux kernels, not too hard to find.
- Offset can be calculated from experiments with different length overruns in `gdb`. 
- Attack string example: `[NOP][NOP][NOP]...[shellcode][ADDR][ADDR][ADDR]...`

An example of a vulnerable program:
```C
char buf[100];
...
gets(buf);
```

This illustrates a case that we try to inject an attack payload into the above program:
```text
		   high addresses 
           +ooooooooooooooo+ |
           +ooooooooooooooo+ |
           +ooooooooooooooo+ |
fp ------> +-----addr------+ |
           +-----addr------+ |
           +---shellcode---+ |
           +-----[NOP]-----+ |
sp ------> +-----[NOP]-----+ |
           +---------------+ |
           +---------------+ v
           low addresses 
```

In this payload, when the `sp` jumps back after successfully completed some stuff, in this case, since `buf[]` was filled with more data than its capacity, the `fp` and `pc` (program counter) now point to: a malicious `shellcode` or redirect the program to `addr`.

