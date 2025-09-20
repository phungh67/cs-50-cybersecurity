#Introduction #beginner

# Definition about Operating systems

> [! What we understood about Computer?]
> Well, normally we make a tie between Operating system and Computer (what we have at home and use it for a variety of purposes including gaming, entertaining and working). But Operating system (refer as **OS** from now on) is larger than that. It provides developers (the guys who live with Monster drink and burgers), users (like us) and CS/CSE students many things to learn, to tinker and to be afraid of.

## 1. The basic anatomy of an OS
---
We should go from the bigger container, to smaller (like the OS). At first, computer system is the one, which needs an OS for operating. Begin with a long history of computation, dated back to the Enlighten era with Ada Lovelace, now we have several super-computes, which are capable of running millions (even billions) of calculation per second. Thus, they still need an OS, for what?

As we see, a computer system is made from many parts:
- Hardware - the basic computing resources: CPU, memory and I/O devices.
- Software - "Something" we are very familiar with, but do they can run directly on a computer system?
- User - Yes, the ones who have something that cannot resolved by themselves, therefore, they force the computer to do their businesses.
So, how do we unite these parts and have a computer "up and running"? We need a glue in-between these things. That is where OS comes to the show.
> [! Definition]
> Operating system is a software, it can act as component of a computer system and also as a resource manager.
> It provides a set of services to system users (collection of service programs) such as: mail, text editor and the most familiar: Graphic User Interface (GUI).
> It also is a shield between user and the actual underlying hardware, not allow user (in somewhat foolish actions) to break the entire computer system (users are not allowed to directly interact with the hardware).
> As a resource manager, it will manage the CPU(s), memory and also the I/O devices.
> Last but not least, OS is a control program (it watches the execution of other programs) to prevent errors and improper use of the computer.


## 2. How does the **OS** manage devices and user?
---
We have known that OS can actually manage all the devices and users. So what exactly it does? There are two points we have to understand (according to this course at Chalmers)/
- We consider that the computer we use have a single core with a single thread (kernel thread) at first, to clearly understand what is "Scheduling" and "Thread" and "Process".
- The CPU of the computer does not work all the time, it also has *idle* period and almost of the time, it will be *idle*.

### 2.1. The OS execution cycle
---
Every time a command is executed, it will be break into smaller processes, which have their tailored missions. For example, a command to list the current working directory can have several step, each is carried by a function: search, sort and list hidden files (if you told it to do so). Each line of code (for this command) will be fetched, executed and halt (to process another one). But most of the time, the OS will in the idle status. It can be only woken up if an interrupt happens and says "Hey, wake up, I have some job for you", probably it will be the timer, each tik-tack is passed, the time will wake the OS and it will tell the CPU to run something that runnable.

So that, the Events (or Interrupts) mark the cycles of OS.

Imagine it will be (somewhat) like that in a conversation between the OS and the interrupt handler(s) (because there are many of them, each will be responsible for a kind of devices, etc...)

> [! Interrupt and OS]
> OS is doing some work...
> Interrupt -> Wakes the OS
> There is a key stroke -> need to print the (just pressed) character on scree.
> OS does that instruction
> And OS comes back to its previous work
> Interrupt -> A bytes copied to the disk
> OS will realize that will mark the complete of a process
> Interrupt...

### 2.2 The system call and the interrupt
---
A process runs in the context of OS will belong to one of two mode: **kernel** or **user**. Each type of mode has its own permissions, advantages, disadvantages and purpose.
- **Kernel** mode: It can run and execute (later) some instructions that are belong to the kernel. In this mode, the process's bit mode is 0.
- **User** mode: Run in the context of user, cannot interact directly with the hardware. If such a process need to access something from the hardware, it must issue the *system call*, a method to trap current instruction and pass it to appropriate handler, which can execute it in the **kernel** mode.

### 2.3 The Timer - a clock for the OS
---
While the CPU is always busy (running something), the OS is mainly idling. But the most important part is: The OS is responsible of managing and ensuring the fairness between processes. If the OS is not interfere, a process can be run infinity (which will waste the power of the CPU, because there is a high chance that instruction has only 1 executable step, while the remaining is I/O stuff, which takes infinite time in POV of the CPU). So how can the OS address this problem? 

> [! The timer]
> Timer is a "clock" for the OS. A timer interrupt (well, an interrupt generated by the timer) can help the OS to enforce **preemptive multitasking** - reclaim the CPU on a schedule

Let's dig deeper into this concept.
A timer generates Interrupt for the CPU - which notifies this and pass that signal to the **time interrupt handler** (as we know, there are many types of them). That signal will wake the OS, which checks the accounting info and deciding whether to change the running process (to be fair, every processes should be executed by the CPU, at least equal to the amount of quantum time).
