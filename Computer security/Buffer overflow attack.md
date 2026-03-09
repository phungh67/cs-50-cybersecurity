
# 1. Introduction

The buffer overflow attack is a form of attacking that takes advantage of the memory layout of a process or more precisely, the memory structure in a program. 

First of all, let's take a review on the programming course:
```C++
#include <string.h>

void sub2(char *str){
	char buf[8];
	strcpy(buf,str)
}

void sub1(){
	char str[] = "Code":
	sub2(str);
}

int main(){
	sub1();
	return 0:
}
```

We have a very convenient object called `Stack` to store all the data in execution for this program. And the `Stack` follows the FILO - first in last out concept. Also, during the execution process of a program, we have several pointers to keep tracking where we currently at, what is going to be executed.

`ip` - instruction pointer, contains the address of instruction we are going to executed.
`sp` - stack pointer, to know where are we in the stack.
`bp` - base pointer or frame pointer to know that start of stack.

In a program, we have this procedure (when calling a function):
- setup function parameters (if any).
- push `ip` of next instruction.
- jump to new function.
- update the `bp` to point to new base. (remember to store the old value!!!)
- update `sp` to the new `bp` to begin a new stack frame.
- setup local variables.
That is, when the executed function returns some value, we just pop the value from the `ip` stack to know what instruction after this function.

So where is the attack? Well, on the stack, we store many variables, thus many data to consider, from address of next memory, to the next instruction we are about to use,... many many important things that you don't want mess them up.

Critically, if the returned address is somehow malformed, we are screw. If it is just simply existed the program with some error code, because of the address that the malformed one tried to access is not valid, it is still fine some how, but with the expertise, they know what exactly the memory looks like, and they prepared something to be executed instead of you intended code.

That is how a buffer overflow attack can cause harm to your system.

# 2. Detail

About the memory structure, we have:
- Argument string (argument to the program).
- Argument pointer (to read those things).
- `argc` - argument count will be the first thing of the stack memory, which is first in last out. And this section grows below, i.e to the `.bss`, `.data` and `.text` section of the data.
- heap memory, the dynamically allocated, with `malloc`, this part is larger than stack, but it grows toward the stack!

So, we have 2 different sections grow toward each other, that is how the problem rose.