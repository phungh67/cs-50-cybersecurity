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

## 2.2. Lock variable

To still maintain the process synchronization, we cannot disable the interrupt. But how to ensure that when a process enters the critical selection section, other cannot, there must be some kind of mechanism to all the others to acknowledge and do not try to interfere.

Implement a global `lock` variable, each time a process tries to enter the critical selection section, it will set the `lock` to `True`, others look into this variable and know that there is "someone" inside. But using a single variable has a takeback, the **race condition**.

**Race condition** is a situation that 2 or more processes try to modify/read a shared value. Imagine if a `procA` comes first and tries to change the `lock` to `True`, but while the `procA` is still changing the variable (because the OS tries ensure **fair schedule**), `procB` can actually read the previous state `false` and changes `true`. This leads to a scenario that 2 processes try to enter the section resulting a **halt** status of the system. Or maybe no process can change the `lock` and the system is stuck again.

## 2.3. Test and Set lock

The next one here is called _Test and Set Lock_. This one is like **Lock Variable**, but it is a bit better because this algorithm requires a little help from the hardware. It solves the problem where one process can read the lock's value before the first process changes the value and thus 2 processes get inside the critical region at the same time. But TSL allows one process to read and change the value of the lock in an atomic way; meaning unless it finishes reading and writing the lock, no other processes can interfere. However, it still doesn't solve a problem where only one process keeps hogging the lock to itself.

## 2.4. Strict alternation

If they have to compete for the lock, then we will specify whose lock this is, so only the owner of the lock can unlock the door and doesn't have to compete for it. This algorithm works with only 2 processes though. But now, no one is competing and no more than specified (which is one by the way) process can stay inside. This sounds better, right? Well, yeah, it is better; a _bit_ better.

But imagine this scenario; process A just finished using the critical region, thus process A gave the key to process B. HOWEVER, process B doesn't want to use the critical region, yet, so it doesn't use the key and the key remain its. Meanwhile, process A now wants to use the key, but it's with process B and unless B uses the key, no other processes can use it. Thus, this algorithm is also problematic and so this doesn't quite work.

## 2.5. Peterson's solution

We're getting more sophisticated with this algorithm. We have more variables which are **turn** and **flag**. This algorithm also works with only 2 processes.

The way it works is first, let's say process A wants to use the critical region, so it sets its own flag to 1 which means process A is interested in entering the critical region, and it changes the turn to its turn (process A's turn). Then process B, which comes a little later, wants to enter the critical region, so it does the same thing; set its flag to 1 and changes the turn to its own. Since process B comes later and changes the turn after process A, the turn, instead of being process A's, it becomes process B's.

The algorithm will first check if both process A's and B's flags are 1 to see if they're both interested. Then, it will check whose turn it is. If they're both interested and the turn is process B's, then process A can enter the critical region first. This is because it knows that whoever comes later can overwrite the turn as its own, and whoever comes first will get its turn overwritten. Then process B which comes later can enter after process A is done. However, if only one process's flag is 1, then that process can just enter the critical region.  

This one is great because every process will have a chance to get inside and no more than 1 process can stay inside simultaneously. Everything seems perfect and all, but is this all?

## 2.6. Sleep and wake up

[[Pintos and the Busy-Wait problem]]