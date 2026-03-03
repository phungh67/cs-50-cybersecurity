For a task set to be executed with RM or DM, there cannot exist an instance of a task execution in the schedule where the worst-case response time of the task exceeds the task's deadline.


We have these tasks below with $C_i$, $D_i$ and $T_i$ as following: [2,5,10], [4,9,12] and [?,8,14] with $\tau_i$

Assume that we have rate monotonic scheduling RM, Derive the largest possible WCET, $C_3$ for $\tau_3$ that all tasks meet their deadline?

We have $$ \forall i, D_i \le T_i; \exists i, D_i \ne T_i $$
PUA (preemptive scheduling) is not valid, RM scheduling, so we have RTA or HPA (Response Time Analysist)
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