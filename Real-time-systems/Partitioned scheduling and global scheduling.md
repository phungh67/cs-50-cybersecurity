
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
With the normal (RM), we have the power of 2 is 1 divided by n with n is the number of tasks in this system, we also don't have n as the multiplier, instead, we have m - number of processors. Comparison with original [[Pseudo-parallel execution]]

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
We have a new element, the m (processor indicate) and also an addition of $c_j$ (because multiple processors also). Also check the original RTA [[Pseudo-parallel execution]]

# 4. Example

# 4.1. Rate-Monotonic-First-Fit

![[Pasted image 20260313204602.png]]

Since these task are used RMFF scheduling, and the relative deadline of each period is equal to its period (which means $D_i$ = $T_i$), so we can apply the Liu and Layland's test with formula of Utilization (PUA).

However, since this system has multi-core processor (3 processors), let call them $\mu_1 \mu_2 \mu_3$ we must find a feasible scenario in which each task is assigned and can run on a processor.

First, apply the formula, we have 

$$
U = \sum_i^n \frac{C_i}{T_i} = 1,79
$$
While the bound for this case is $m(2^{\frac{1}{2}} - 1)  = 1.24$ so that the test failed, but because it is only a sufficient test, then we must do the RMFF algorithm test (check for each processor manually).

For the first processor $\mu_1$ it is currently idle, so that it can accept a task, with the Liu's original formula, $n(2^{\frac{1}{n}}-1)$ , still fit

Try with all, we have 6 tasks can be successfully scheduled on 3 processors.











































