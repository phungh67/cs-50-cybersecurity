
# A. Real-time System Programming with `TinyTimber`kernel

`TinyTimber`provides a built-in real time clock to measure the execution time, for a task to be scheduled,... serve the "real time" purpose.

The resolution of this clock is $10\mu s$ - which means:
- If want to measure any time period that is smaller than this resolution, a directly measurement is not possible.
- The best way to achieve this is via calculation. For example, if we know that the execution time of this task is about 100 times lower than the base resolution, we should put that task inside a loop of 100 times (or more if possible), after that, we use another operation to measure the start time and stop time of the loop.

Operations related to timing purpose:
- `CURRENT_OFFSET()` returns the current timeline from the current baseline.
- `T_RESET()`used to bookmark the current baseline. The variable used to store this is an object from class `Timer`.
- `T_SAMPLE()`measures the time duration from the bookmark to the baseline of a later even.

So the different is: while current offset calling returns the current timeline from current baseline, the sample calling will return the time duration from the bookmark to the baseline of a later event.

Which means the current offset is purely time-checking, while the sample is more used for calculating purpose, since it depends on the baseline of an event (as stated in the slide, lecture 6, page 9, Real Time System course).

>[!Notes]
>There is difference between periodic and sporadic task.
>For periodic task, it will arrive at a predictable manner, well defined period, well defined arrive time (but apparently, completion time and whether it satisfies deadline is are not guaranteed).
>On the other hand, the sporadic task is still predictable, but arrival time is not guaranteed as we see in the periodic taks.

# B. Time-related concerns and phenomenons

Normally, a task can arrive at a identified time, but the total time of execution, waiting and periodic waiting can cause some delayed between 2 instances of the very same task.

For example: a periodic task is a task that we can surely estimate the arrival time and can definitively know that there is always an instance of that task appears every $\tau$ time (the period). But we have a new thing to consider: "systematic time skew".

>[!Definition]
>Systematic time skew is a phenomenon caused by the "prolong" of the waiting time between 2 sequentially instances of the very same task. Because each task have an action time (or execution time) $\delta_{action}$ and maybe the "administration time" - the time for condition evaluation, constructing the loop, take out the function from the stack,... $\delta_{loop}$, hence, making the gap between 2 instances is now the sum of period time, execution time and loop time. Later, the more instances have come, the longer the waiting time between is.

In the `TinyTimber`kernel, there are several methods that implement (and address) these phenomenons. For example, the `AFTER()` function delays the execution of a method to the earliest offset (so you don't have to worry about the "administration" cost) - it's not like the administrator cost is reduced to zero, but it is more like that you can predict and calculate the execution, completion time better, because the task would not be executed util all the administration stuff was completed.

# C. Priority inversion

In computing, programming and even real-time controlling, the shared resources are a very hard-to-deal problem.
A shared resource is a resource that can be accessed by multiple instances (tasks, objects,...) and the lock (mutex) mechanism is a way to ensure that it can only be used once at a time to avoid race condition.

But in a scenario, we have three tasks: High, Medium and Low. Normally, the Low arrived first, then enter the critical selection region, accessing the shared resource. After a while (low is still executing and still holding the resource), apparently the high task will be prioritized to be executed. On the other hand, since the resource is still held by low, and because low was preempted before it could release the key, the High was block, and the low continued its execution. Now a Medium task came, and the low would be halted. In conclusion, the High task was not only blocked by low because a shared resource was not released, but was also interrupted by lower priority task's execution (Medium).

## Priority Inheritance Protocol

Assume that a lower task currently blocks several higher tasks (due to it still holds the shared resource) - it will temporary inherits the highest priority in all blocked tasks. This solution aim to not prolong the execution time caused by preemption. The disadvantages: **deadlock** and **chained blocking** - in which the highest priority task can be blocked once by every other task executing on the same processor (which still makes the high task not-so-high anymore).

## Priority Ceiling Protocol

Each resource is assigned a priority ceiling equal to the priority of the highest priority task that can lock it. A task can only able to enter a critical region only if its priority is higher than all priority ceilings of the resources currently locked by a task other than it - no deadlock. A task that can only enter critical region must be higher-priority than the current owner.

When a task blocks one or more higher priority tasks it temporarily inherits the highest priority of the blocked task - so the task can only be blocked by higher task, at most blocked once during the critical region time.


# C. Execution-time analysis

A Worst case execution time `WCET`estimate must be: pessimistic but tight $0 \le (Estimated WCET) - Real(WCET) \le \epsilon$  with $\epsilon$ is very small compared to real result.

**Pessimistic**: to make sure the assumptions made in the schedulability analysis of hard real-time tasks also apply at run time.
**Tight**: to avoid unnecessary pessimism in the schedulability analysis, which could cause the feasibility tests to be too inaccurate to be useful.

Dhall's effect
>[!Definition]
>This phenomenal means that the task will missed the deadline no matter how many core you add to the processor. Why? Because when light tasks and heavy tasks are mixed together for execution, because the dynamically scheduler will favor the light tasks and they can be executed, completed in a very rapid time, but for heavy task, the task in which the execution time is nearly the same with the period, the scenario is not good. Because all light tasks are prioritized to be executed first, the heavy task will be shifted a little, resulted in missing the deadline.

As stated in the slide:

> [!Slides] Dhall's effect
> Dhall's Effect describes a paradoxical situation in global multiprocessor scheduling (like Global RM or Global EDF) where a task set can be unschedulable on $M$ processors even if the total system utilization is incredibly low (approaching exactly 1, meaning almost all processors are completely idle).
> It happens when you mix many "light" tasks (tasks with tiny execution times and very short periods/deadlines) with one "heavy" task (a task where execution time is almost equal to its period, $U \approx 1$). Because global schedulers prioritize based on short deadlines/periods, the light tasks are given top priority. They constantly bounce around the processors, repeatedly preempting the heavy task. The heavy task is starved of the continuous CPU time it desperately needs and misses its deadline, even while other processors sit empty.
