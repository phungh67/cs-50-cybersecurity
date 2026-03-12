
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
$$  R_i^{n+1} = C_i + \sum_{v_j} [\frac{R_i^n}{tau_j}] * c_j $$
To be short, we take every task, to inspect the Response Time $R$ which is calculated by take the execution time of the current task $c_i$ plus the total time of interference tasks (whose priorities are far higher than current task). 

In the interference element, we take the ceiling of the previous Response Time, divided by the period of higher task then multiply the result with execution time of that higher one.


## 4. An example where Liu & Layland's is not enough

![[Test-is-not-enough.png]]

Even with the result from formula, with the Hyper Period Analysis to draw the time table, the task 3 was failed. Then we must move to Response Time Analysis. But before that, we have another type Deadline-monotonic

>[! Deadline monotonic]
>Priority of a task is determined by the urgency, the task with the shortest relative deadline receives the highest priority.

It can be considered as a generalization of the rate-monotonic scheduling, because in this case, RM is a special scenario where $D_i$ = $T_i$

This kind provide exact feasibility test , optimal among static task with constraint: $D_i$ $\le$ $T_i$



We have these tasks below with $C_i$, $D_i$ and $T_i$ as following: [2,5,10], [4,9,12] and [?,8,14] with $\tau_i$

Assume that we have rate monotonic scheduling RM, Derive the largest possible WCET, $C_3$ for $\tau_3$ that all tasks meet their deadline?

We have $$ \forall i, D_i \le T_i; \exists i, D_i \ne T_i $$
PUA (Processor Utilization Analysis) is not valid, RM scheduling, so we have RTA or HPA (Response Time Analysist and Hyper Period Analysis)
RM pros: $\tau_1$ > $\tau_2$ > $\tau_3$ (since $T_1$ < $T_2$ < $T_3$)
$$  R_i^{n+1} = C_i + \sum_{v_j} [\frac{R_i^n}{tau_j}] * c_j $$
(with $R_i^0=C_i$) so we have $R_1$ = $C_1$ = 2 $\le D_1$ =5 (OK)
With $R_2$ we have $R_2^0$ = 4,  $R_2^1$ = 6 and $R_2^2$ = 6, convergence with $R_2$ = 6 $\le D_2 = 9$, acceptable

For task 3, we have 
$$ R_3^{n+1} = C_3 + [\frac{R_3^n}{tau_1}] * c_1 +  [\frac{R_3^n}{tau_2}] * c_2 $$
Try with 1, since the Deadline for 3 is 8, we would have 1 is sufficient since response time for task 3 is convergence within the iteration 2 (1 and 2). Try with 2, we have the exactly result $C_3=2$ is the maximum because response time $R_3$ converged with iteration 1 and 2, with value of 8, equals to the deadline, hence we cannot push further.

DM pros: $\tau_1$ > $\tau_3$ > $\tau_2$ (since $D_1$ < $D_3$ < $D_2$)
$$ \forall V_i; D_i \le T^{min} = 10 => R_i^{n+1} = C_i + \sum_{v_j} [\frac{R_i^n}{tau_j}] * c_j $$
Because of min, we have $[\frac{R^{max}}{T^{min}}] = 1$ 
$R_1 = C_1 = 2 \le D_1 = 5$ 
$R_3 = C_3^{max} + C_1 \le D_3 => C_3^{max} = 6$
Also have $R_2 = C_2 + C_1 + C_3^{max} \le D_2$
We have to choose the smaller value, if not, task 2 can not make its deadline (because, these trio must be schedulable and meet their deadline respectively).


Consider a real time system with periodic task s and a run time system that employs scheduling on uniprocessor. All tasks arrive the first time at time 0.
We have the table with $C_i$, $D_i$ and $T_i$. Using EDF for $C_3$ where $5 \le C_3 \le 7$, is it possible, why or why not
[2,5,5], [3,6,10] and [?, 2$C_3$, 20]
For all task, we have the deadline is less that the period, and also, exists one deadline that different with the period. Furthermore, EDF, so PUA (process utilization analysist) is not valid. Dynamic priority leads to PDA or HPA