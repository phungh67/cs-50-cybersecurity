
>[!Partitioned Scheduling]
>If all tasks are assigned using the Rate-Monotonic-First-Fit (RMFF) algorithm, then all tasks are schedulable if the total task utilization does not exceed 41% of the total processor capacity.

>[!Global scheduling]
>If tasks with the highest utilization are given highest priority and the remaining tasks are given RM priorities according to RM-US, then all tasks are schedulable if the total task utilization does not exceed 33.3% of the total processor capacity.

# 1. Basic formula

Since the task is executed on multiple processor, with this partitioned approach, each task is considered as a single task in the single processor system. Because each core in this processor has its own queue to put the waiting task into.

Because of that, we still use the basic feasibility test, the Process Utilization Analysis formula, but this time we have a new right hand-side.

$$
U  = \sum_i^n\frac{C_i}{T_i} \le m(2^{\frac{1}{2}}-1)
$$
With the normal (RM), we have the power of 2 is 1 divided by n with n is the number of tasks in this system, we also don't have n as the multiplier, instead, we have m - number of processors.

So with each m = 1 (each separate processor), if the total utilization of all tasks which would be scheduled on that processor was less than 41% (square root of 2 minus 1), we are safe.

# 2. RM-US global scheduling

We have a new formula $RM-US[m/(3m-2)]$ due to:
- Dhall’s effect:
	- With RM, DM and EDF, some low-utilization task sets can be non-schedulable regardless of how many processors are used. Thus, any utilization guarantee bound would become so low that it would be useless in practice.
	 - This is in contrast to the single-processor case, where we have utilization guarantee bounds of 69.3% (RM) and 100% (EDF).
- Hard-to-find critical instant:
	- A critical instant does not always occur when a task arrives at the same time as all its higher-priority tasks.
	- This is in contrast to the single-process

In conclusion, new formula (still based on Process Utilization Analysis):
$$
U = \sum_i^n \frac{C_i}{T_i} = m * \frac{m}{3m - 2}
$$

# 3. Response Time Analysis
$$
R_i = C_i + I_i = C_i + \frac{1}{m} \sum_{\forall j \in hp(i)} (\lceil\frac{R_i}{T_j} \rceil.C_j + C_j) \le D_i
$$
We have a new element, the m (processor indicate) and also an addition of $c_j$ (because multiple processors also).

# 4. Example

There are 6 tasks with $C_i$, $D_i$ and $T_i$ as usual (execution time, deadline and period). But in this, we only need PUA - Process Utilization Analysis so $D_i$ is not necessary. We have total of 3 processors. The given values are [2,10], [10,25], [12,30], [5,10], [5,20], [7,100]

The utilization can be calculated by taking the execution time divided by the period (the actual meaning time within a period). So we have array of $U_i$ = {0.2, 0.4, 0.4, 0.5, 0.4, 0.07}

We have 
$$ PUA (U) = \sum \frac{C_i}{T_i} \le m(2^{\frac{1}{2}}-1) $$
With the given values, test failed so only sufficient with the bin-packing algorithm.

So with RM priority, the lowest period get the highest, because 2 tasks are equal, but no matter what we choose. So we take task 1 first. $$\tau_1 = \tau_4 > \tau_5 > \tau_2 > \tau_3 > \tau_6 $$
But since processor is limited, so take the execution time as the tie-break.
To test if a task can enter a processor $\mu$, we check the formula with $PUA$ as above, but with the $m$ equals to the sequence number (1,2,3,...).

Do the math, $\tau_1$ is good, $\tau_2$ also good to, but $\tau_5$ is not (in p1). Because, 1 and 4 are on the same level, so they go together at the same time. After that, we try to fit task with processor (First Fit).


Consider that the task set below for a system using global scheduling on m=3 processors. Assume that task priorities are given according to RM-US[m/(3m-2)]

[1,7], [4,19], [9,20], [11,22]

With RM-US scheduling, 4 tasks and 3 processors

$$ U = \sum \frac {c_i}{\tau_i} \le \frac{m^2}{3m-2} | m * \frac{m}{3m - 2}$$
Do the math, test provided that it was not sufficient, so must try response time analysis (RTA) on multiprocessor

We have the formula for RTA (but applied for multi processors)
$$ R^{n+1}_i = c_i + \frac{1}{m} . \sum ([\frac{R^n_i}{\tau_j}]c_j + c_j) | R_i^0 = c_i$$
If $U_i$ $\le$ $\frac{m}{3m-2}$ it would be top priority otherwise RM priority. Hence $\tau_3$ and $\tau_4$ are considered as top with $\tau_1$ and $\tau_2$ follow after.

With n=4 tasks and 3 processors. interference can only be sufficient by the n-m=1 lowest-priority task ($\tau_2$) The others are trivially schedulable on one processor each. So we only need to do RTA for $\tau_2$
ed













































