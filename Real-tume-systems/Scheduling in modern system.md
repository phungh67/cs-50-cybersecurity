## Example 1: Cyclic executive
**Problem**: Consider a real time system with two specific periodic tasks that should be scheduled using a time table. The parameters for two tasks are given below. Both tasks arrived at time 0
a. Construct a time table for the execution of the two tasks. These tasks are allowed to preempt each other.
b. Does your schedule constitute the best possible schedule or does there exist a superior one.
  C D T (execution time, deadline and period)
T1 2 5 5
T2 4 7 7


a. we have this time table, which can be understood as:
( task1-1[0,2], task1-2[2,6], task1-3[6,8],...)

![[Scheduling diagram of tasks.png]]

**Problem**: Having 3 task, which are [1,7,7]; [1,14,14] and [4,18,18] with execution time, deadline and period respectively. Can we guarantee the schedulability of the task set using the RM scheduling algorithm? (RM is rate-monotonic which is, task's priority is inversely proportional to its activation period).


Start with processor utilization allowance (PUA):
$$U = \sum_{i=1}^{n} \frac{C_i}{\tau_i} \le n(2^{1/n} - 1)$$
In which, t is for period (we use Greek character) and n is the sum of task.
In this case, we have 
$$U = \frac{1}{7} + \frac{1}{14} + \frac{4}{18} = 0.44$$
We also have the right hand side equals to 0.78
So this case, tasks can be schedule (since the PUA is sufficient)

What if we have another task 4 with [x,100,100]. What is the maximum allowance value of the x, that is schedulable for RM with "Liu and Layland's" utilization set (the formula above)

We have $$ \frac{1}{7} + \frac{1}{14} + \frac{4}{18} + \frac{x}{100} \le n(2^{1/n} -1)$$
Replace with value, we have
$$ 0.44 + \frac{x}{100} \le 0.76 $$
In fact, x could possibly be higher because this is only a hypothesis.

**Problem** we have 3 tasks with the RM scheduling model. [6,12,16], [4,13,14] and [20,70,75]. Perform response time analysis for the given task set. The convergence response time should be calculated for each task in the set regardless of whether the repose time analysis for that task fails or not. Also, it is safe to assume that $R_n^0 = C_n$
$$ R_i = C_i + I_i $$ In which the $I_i$ is the interference from higher priority task, which leads to
$$ R_i^{n+1} = C_i + \sum_{v_j} [\frac{R_i^n}{tau_j}] * c_j $$
With the given values, we have $\tau_2 < \tau_1 < \tau_3$ (task has smaller period will have the higher priority)
$$ R_2 = C_2 = 4 \leq D_2 = 13 $$
(because 2 has the highest priority over three)
$$ R_1^{n+1} = C_1 + [\frac{R_1^n}{T_2}].C_2 = C_1 + [\frac{R_1^n}{14}].4 $$
So we have $R_1^1$ and $R_1^2$ also have the same value of 10, so it is converged, and of course, less than 12 (the deadline for $\tau_1$). Similarity, we have
$$ R_3^{n+1} = C_3 + [\frac{R_3^n}{T_2}].C_2 + [\frac{R_3^n}{T_1}].C_1$$
Convergence with task 3 happens in 64, less than deadline, so it passed to.

These task set are schedulable (under RM algorithms). As long as all the tasks have the convergence time less than its respectively deadline.
So "Test succeeds and it exact"