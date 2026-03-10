
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

# 3. Exploited

The most common way is to control the buffer, point it to the wanted code piece. The attacker had setup some instructions in the "fake" return address of the code.

# 4. Counter

Best method first: defensive programming, always remember to check the boundary before performing any computation, only allow a fixed length of input.

Canary is a quite common method. That is we put a variable named "Canary" in between the return address, for example, between the local variables and frame pointer section. So that if any attackers tried to overflow some value within the variables and try to overwrite the return value, the value of canary was changed also. And to do so, the canary must pass the pre-check.

With that, we can make sure that there is no illegal modification during the execution.

But, attacker always wants to enhance the attack method. They even can try to exploit the code to by pass the canary by completely not touching it.

Secondly, we have the data execution prevention - we separate memory zone, like, any zone can not be both writable and executable. So that even if the attackers can somehow malformed the code to run instructions on their own, it cannot be executed. But once again, the adversaries can still find somewhere to attack.

In the system, we have a lot of code in the libraries (the system one). And this time, attackers try to drive these program to return to system libraries (C), but assume there are 3 things:
- Can manipulate the code pointer.
- The stack must be writable.
- Know the address of a "suitable" library.

Address Space Layout Randomization - try to implement the the layout of memory in a randomly way, try to force the attackers in a blind trap, that they really don't know how to guess or how to predict which part in memory they should attack:
- Randomized the stack and the start address of the code (stack and base).
- But let's assume that functions, variables are still at the same relative offset from the start address. (i.e, a function with 6 params still takes 6 * 32 bytes?).
- So that, only need a single code pointer.

# 5. Summary

In conclusion, the adversary's intention is: trying to inject malicious things into the code. If it wasn't worked, trying to change the control flow of the program instead to make it execute "malformed" instructions.

Also, for overflowing stuff:
- Variables can be overwritten as a result of this action (because they lie on the same stack).
- Other addresses (function, control method,...).
- Etc

```C++
get_medical_info() {
	boolean authorized = false;
	char name[10];
	read_from_network(name);
	
	if (authorized)
		show_medical_info(name)ö
	else
		printf("You are not allowed to do this.\n");
}
```

In the above example, an overflow from variable `name` can cause the value of `authorized` to change. Because it is only `false` if the value of that address equals to zero, otherwise, it will be `true`, then medical data (very sensitive) can be exposed.
