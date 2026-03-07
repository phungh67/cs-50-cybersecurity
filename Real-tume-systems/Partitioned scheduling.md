
RMFF Scheduling - Rate monotonic First Fit

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













































