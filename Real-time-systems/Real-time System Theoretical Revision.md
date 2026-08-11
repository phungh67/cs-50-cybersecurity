
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

With `TinyTimber` kernel, it uses Earliest-Deadline-Priority `EDF`, which can be further leveraged by using method like `BEFORE()`or `SEND()`. In this case, `BEFORE()`specify a deadline, then the method should be completed before that deadline. On the other hand, the `SEND()`method allowed user to invoke a method from a predefined baseline, and requires it to complete before a deadline.

The `ASYNC()`and `SYNC()`are two fundamentally building blocks of the kernel, and as their names, they invoke task asynchronously and synchronously. To achieve concurrent programming, the `ASYNC()`method should be used, since the `SYNC()`method will block the execution till the callee returned some result (which can result a very long waiting time).

>[!Notes]
>There is difference between periodic and sporadic task.
>For periodic task, it will arrive at a predictable manner, well defined period, well defined arrive time (but apparently, completion time and whether it satisfies deadline is are not guaranteed).
>On the other hand, the sporadic task is still predictable, but arrival time is not guaranteed as we see in the periodic taks.

# B. Time-related concerns and phenomenons

Normally, a task can arrive at a identified time, but the total time of execution, waiting and periodic waiting can cause some delayed between 2 instances of the very same task.

For example: a periodic task is a task that we can surely estimate the arrival time and can definitively know that there is always an instance of that task appears every $\tau$ time (the period). But we have a new thing to consider: "systematic time skew".

>[!Definition]
>Systematic time skew is a phenomenon caused by the relative delay when trying to making waiting time during real time programming. Relative delay caused the object/method to wait with a "piece of time" from that point - hence, due to administrator cost of constructing conditional code, the delay might be prolonged for every subsequent instances. This can be solved by using absolute delay, with method `SEND()`of `AFTER()`in the `TinyTimber` kernel, enforces the delay based on a pre-defined baseline.

In the `TinyTimber`kernel, there are several methods that implement (and address) these phenomenons. For example, the `AFTER()` function delays the execution of a method to the earliest offset (so you don't have to worry about the "administration" cost) - it's not like the administrator cost is reduced to zero, but it is more like that you can predict and calculate the execution, completion time better, because the task would not be executed util all the administration stuff was completed.

**Periodic tasks** - the time duration between 2 subsequent tasks is exactly $T_i$ 
**Sporadic tasks** - the time duration between 2 subsequent tasks can be equal or greater that $T_i$

# C. Priority inversion

In computing, programming and even real-time controlling, the shared resources are a very hard-to-deal problem.
A shared resource is a resource that can be accessed by multiple instances (tasks, objects,...) and the lock (mutex) mechanism is a way to ensure that it can only be used once at a time to avoid race condition.

But in a scenario, we have three tasks: High, Medium and Low. Normally, the Low arrived first, then enter the critical selection region, accessing the shared resource. After a while (low is still executing and still holding the resource), apparently the high task will be prioritized to be executed. On the other hand, since the resource is still held by low, and because low was preempted before it could release the key, the High was block, and the low continued its execution. Now a Medium task came, and the low would be halted. In conclusion, the High task was not only blocked by low because a shared resource was not released, but was also interrupted by lower priority task's execution (Medium).

## Priority Inheritance Protocol

Assume that a lower task currently blocks several higher tasks (due to it still holds the shared resource) - it will temporary inherits the highest priority in all blocked tasks. This solution aim to not prolong the execution time caused by preemption. The disadvantages: **deadlock** and **chained blocking** - in which the highest priority task can be blocked once by every other task executing on the same processor (which still makes the high task not-so-high anymore).

## Priority Ceiling Protocol

Each resource is assigned a priority ceiling equal to the priority of the highest priority task that can lock it. A task can only able to enter a critical region only if its priority is higher than all priority ceilings of the resources currently locked by a task other than it - no deadlock. A task that can only enter critical region must be higher-priority than the current owner.

When a task blocks one or more higher priority tasks it temporarily inherits the highest priority of the blocked task - so the task can only be blocked by higher task, at most blocked once during the critical region time.

## Critical Instant

Refer to the scenario in which during a task's arrival, the response time of a given task is maximized. Applied for both single and multiple processors system.

In the preemptive schedule in single processor, the most notably demonstration is that tasks arrive at the same time, resulting the lower priority task might take longer time to response (because the system favored the high priority tasks) but in the multiprocessor scheduling, this scenario may not be applied.

## Deadlock

Deadlock is a phenomenon that can happen when 2 object both wait for other object to release the requested shared resources. 
There are two task $\tau_1$ and $\tau_2$ with 2 resources $R_1$ and $R_2$. The priority is high for task 1 and low for task 2. So apparently, task 2 will be preempted if task 1 is ready. When there is no task, 2 can be run and lock resource 2. Then 1 became ready and run, then run first because priority. It locked resource 1 and tried to acquire resource 2 but since $\tau_1$ was preempted before releasing, it would be blocked. Under `PIP`, task 1 would become high priority, continued to executing and tried to acquire resource 1, which is held by $\tau_2$, hence created a deadlock.

## Chain lock

Very easy to misunderstand this concept with the deadlock. While deadlock may refer to a situation where 2 object circular wait and request each other to unlock a necessary object, chain lock is about a high priority task was reverted to the lowest one.

We have four tasks: 1,2,3 and 4 with 3 resources (shared) a, b and c. The priority is increasing from 1 to 4. 1 run first, lock a, then preempted by 2, lock b and then preempted by 3 and lock c. At that time, 4 run, but cannot acquire c since it was locked by 3, then under `PIP`, 3 was elevated to same level as 4, but also locked by 2, then 1, then finally, 1 inherited the highest priority as same as 4, run, then release a, then 2, then 3 and finally 4. 

4 is the highest, but turned out locked by 3, 2, and 1 (chain lock). Then executed last in the pipeline.

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

# D. Scheduling analysis

The feasibility test produces only 2 outcome and they are always binary, "True" or "False". But the conclusion about the schedulability of a given task set depends on the test: sufficient, necessary or exact (sufficient and necessary).

- **Sufficient test**: a "Yes" outcome means the task set will be scheduled in the processor, but a "No" outcome is insufficient to conclude that these tasks could not be scheduled in the said processor. Further investigation is needed.
- **Necessary test**: a "No" outcome proves that the given task set could not be scheduled in the processor, but a "Yes" is also not enough to prove the scheudulability.
- **Exact test** is a combination of both necessity and sufficiency, which provides unified answer, a "Yes" is a yes and a "No" is a no.

In the term of feasibility test, there are several methods that are widely used:
- Hyper period analysis - HPA for short: in an existing schedule, no task execution may miss its deadline. The disadvantage comes from the drawing of long long execution time because we must analyze them at least $T$ while $T$ is the least common multiplier of these tasks.
- Processor utilization analysis - PUA: the fraction of processor time that is used for executing the task set must not exceed a given bound (the Liu and Layland formula).
- Response time analysis - RTA: the worst-case response time for each task must not exceed the deadline of the task (commonly used with RM or DM system).
- Processor demand analysis - PDA (commonly used in the EDF system) the accumulated computation demand for the task set under a given time interval must not exceed the length of the interval.