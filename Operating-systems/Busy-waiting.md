Definition of busy waiting, what is it and how to avoid it (from naive approach to more complex ones).

#OS #scheduler

# 1. Definition
>[! What-is?]
>Busy waiting is a method of the process synchronizing in which a process/a task waits and constantly checks for a condition to be satisfied before proceeding with its execution.

So, to be clear,  you have 2 processes, `procA` and `procB`. They are both able to access a shared resource, called `shared`. When the `procA` is inside the critical selection state (well, a state that `procA` uses `shared` and it can make some modifications or something that must be consistency), the `shared` can not be used by any other processes. Hence, `procB` must wait the `procA` complete before executing.

As mentioned above, this is a simple technique that ensure the process synchronization. This disallow the 'race-condition' between two or more processes. But it is clearly to see that, the continuously checking for the 'free' of `shared` wastes the CPU power, also wastes power. There must be something we can do to optimize this process, allow the CPU to do some meaningful jobs instead of checking and asking "is it free?".

# 2. Early approach.

The OS normally does not care about what is the CPU doing, it simple wakes up and tell CPU that "Hey, you should change your process (if it is running enough) to ensure the fairness". And how does the OS do that? It relies on the `timer`, which acts as an alarm to periodically wakes the OS and tell it to check the CPU. So, the busy waiting of course relies on this too. Each *tick* passed (the time measurement for atomic time of the CPU), the interrupt will wake the CPU, the CPU will tell the thread to wake up and check for time (if the time passed is greater than the `sleep` time, it will enter the `READY` state).

## 2.1. Disable the Interrupt

This is the simplest solution, you don't have to care about buys-waiting if you got rid of the original: `interrupt`.
But since it was disabled, once a thread is running, it will run to the end of its execution. And if that thread needs something it does not have and must wait for some delivery, disabled the interrupt will make your system enter the **hanging-forever** status.

