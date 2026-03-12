
![[Pasted image 20260312155156.png]]

# 1. Task execution

- Static assignment: we have **Rate monotonic** and **Deadline monotonic**
- Dynamic assignment: we have Earliest-Deadline-First (EDF).


In term of priority, we have:
>[! RM] 
>Rate monotonic - priority determines by the task frequency (rate), the task with the highest rate is also the task with the shortest period.

This approach has these properties:
+ Static priorities (only take the period in account).
+ Sufficient test can be done in liner real time.
+ Exact feasibility test in an NP-complete problem.
+ Optimal among all scheduling algorithms that use static priorities for implicit-deadline task.

>[! Earliest - Deadline - First]
>As it name states all, the task with the closet deadline will win the ticket to be executed.

Properties:
- Dynamic priorities, since task can be preempted because the decision was made after a state changed, and the task with absolute deadline is earliest in time receives highest priority.
- Exact feasibility test can be done in liner real time.
- Also this test in general is a NP-complete problem.
- Optimal among all scheduling that uses dynamic task priorities.

For a task set to be executed with RM or DM, there cannot exist an instance of a task execution in the schedule where the worst-case response time of the task exceeds the task's deadline.

# 2. Conduct the test

With the HPA (hyper period analysis - check [[Hyper Period Analysis (HPA)]]), just draw since the period is quite short? and if EDF, use it too.

But with this time, Processor Utilization Analysis (PUA), first we have the total CPU utilization can be calculated as:
$$
\sum_i^n \frac{C_i}{T_i}
$$
With $C$ is the execution time of the task and $T$ is the period.

## 2.1. Sufficient condition for RM (not include necessary)

In this case, we also have a new algorithm called "RM" stands for Rate-monotonic priority, that is, based on task's period, the task with shorter period (executed more frequently) will have higher priority.

We also have a formula, the Liu and Layland's feasibility test
$$
\sum u_i \le n(2^{\frac{1}{n}} - 1)
$$

In this formula, U is the CPU utilization, while $n$ represents the number of tasks.

Taking this result and perform some limitation on it, it show that, when n goes to infinite, the bound of this formula is $ln2$ which means, for any given set of task that the total utilization does not exceed $ln2$ is always schedulable.

# 2.2. With EDF

It is simpler than RM, as long as the $\sum_i^n \frac{C_i}{T_i}$ is less than 1, test is passed !


![[Common-scheduling-algorithm.png]]


# 3. Is the test enough?

No, just test is not enough, the formula is just the sufficient test, a no cannot mean that there is not possible algorithm for the task set, and also, the "Liu and Layland's", which draws from the Processor Utilization Analysis (PUA), suitable for BOTH static and dynamic, so there is also something that were not covered.

So we have another test Response Time Analysis (RTA).
$$  R_i^{n+1} = C_i + \sum_{\forall \tau_j} [\frac{R_i^n}{T_j}] * c_j $$
To be short, we take every task, to inspect the Response Time $R$ which is calculated by take the execution time of the current task $c_i$ plus the total time of interference tasks (whose priorities are far higher than current task). 

In the interference element, we take the ceiling of the previous Response Time, divided by the period of higher task then multiply the result with execution time of that higher one.


## 4. An example where Liu & Layland's is not enough

![[Test-is-not-enough.png]]

Even with the result from formula, with the Hyper Period Analysis to draw the time table, the task 3 was failed. Then we must move to Response Time Analysis. But before that, we have another type Deadline-monotonic

>[! Deadline monotonic]
>Priority of a task is determined by the urgency, the task with the shortest relative deadline receives the highest priority.

It can be considered as a generalization of the rate-monotonic scheduling, because in this case, RM is a special scenario where $D_i$ = $T_i$

This kind provide exact feasibility test , optimal among static task with constraint: $D_i$ $\le$ $T_i$

>[! Notes]
>If for all tasks in the task set, if there is any task (or all) has $D_i \le T_i$ the Process Utilization Analysis (the formula from Liu and Layland) cannot prove the feasibility for the task set to be scheduled. The Response Time Analysis must be used.


### Example

![[scheduling-example-rm.png]]

In this case, we have a set of 3 tasks and the goal is:

>[! Question a]
>Find the largest integer value of $c_3$ such that all tasks meet their deadline.

We have $D_i \le T_i \forall i$ so that the **Process Utilization Analysis** is not applicable. The only method to determine this task set can be scheduled or not is **Response Time Analysis**.

Because this system is RM-based, so the priorities for all these tasks will be $\tau_1 < \tau_2 < \tau_3$

With the formula to calculate the response time $R$, for each task, we have:

- For $\tau_1$ we have $R_1 = C_1$ and equals to 2, less than deadline, good.
- For $\tau_2$ since it has lower priority than $\tau_1$, the response time will be: $$ R_2^{n + 1} = c_2 + \lceil \frac{R_2^n}{c_1} \rceil * c_1 $$
	Luckily, with this one, first $R_2⁰$ = $c_2$ so that we have $R_2¹$ = 6 and also $R_2²$ = 6, convergence happened and it also less than the $D_2$.
- For $\tau_3$, the generic way is to "backtrack", first, choose $c_3 = 1$ then calculate, if the convergence is still less than $D_3$, increase the value of $c_3$. And of course, we pick the maximum value of it.

>[! Question b]
>Also the same requirement with above question, but this time, the system uses Deadline monotonic.

Since Deadline monotonic is applied, we have $\tau_1 < \tau_3 < \tau_2$

Same as before, we still have task 1 as the highest priority and it totally fine.

With task 3, because we want the largest possible value of $c_3$, since the largest possible value is as the same as the deadline, so applying the formula of Response time, we have $R_3^{max}$ = $D_3$ then we have $c_3^{max}$ equals to 6.

But from task 2, we also have another value (since we also want the maximum value of task 3, and task 2 is interfered by task 3, then task 2 response time should be max as well). With that we have $c_3^{max}$ equals to 3.

Pick the minimum of it, we have 3 (because task 2 must converge to meet the requirements).