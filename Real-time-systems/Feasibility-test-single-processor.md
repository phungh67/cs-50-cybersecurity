
As told before, even the Processor Utilization Analysis is pass, it does not mean that the task set can be scheduled. Because there are some worst cases that several task (or just one task) cannot meet it deadline. To prove that these tasks are not schedulable, need some test to prove it.

# 1. Processor demand

The processor demand for a task $\tau_i$ in a give time interval [0, L] is the amount of processor time that the task needs in the interval in order to meet the deadline that fall within the interval.

Let $N_i^L$ represents the number of instance $\tau_i$ that must complete execution before L.

The total processor demand up to L is $$
C_P(0, L) = \sum_{i=1}^n N_i^LC_i $$
Of course, we must count all task with its respectively execution time $c_i$

In the other hand, we can express $N_i^L$ as:
$$ N_i^L = \lfloor \frac{L - D_i}{T_i} \rfloor + 1 $$
Then, plug it in to the final formula:
$$
C_P(0, L) = \sum_{i=1}^n (\lfloor \frac{L - D_i}{T_i} \rfloor + 1) * C_i 
$$
# 2. Condition for feasibility test

A sufficient and necessary condition for EDF scheduling (exact) of synchronous task sets, for which $D_i \le T_i$ is
$$ \forall L: C_p(0, L) \le L $$
where $C_P(0, L)$ is the total processor demand in [0, L]

Also these condition (assumptions) must be present for the test to be valid:
- All tasks are independent - there must not exist dependencies due to precedence or mutual exclusion.
- All tasks are periodic or sporadic.
- All tasks have identical offset.
- Task deadline does not exceed the period.
- Task preemptions are allowed.

# 3. How many intervals are enough?

We consider two things, first it the least common multiply for all periods
$LCM(T_1, T_2, ... T_i)$ and the bound of the synchronous task set by Baruah, Rosier and Howell: $L_{BRH} = max (D_1, ... D_i, \frac{\sum_{i=1}^n(T_i - D_i)U_i}{1 - U})$

In summary, for every synchronous task set, we can use the LCM, but if we have U $\le$ 1, we can use the least upper bound $L_{BRH}$ so that we have $L_{max} = min(L_{BRH}, L_{LCM})$

Summarize for single-processor scheduling
![[feasibility-test-with-several-type-scheduler-algorithm.png]]

# 4. Example

>[! Notes]
>For EDF process-demand, there cannot exist an interval length of L in which this task set can be scheduled if the processor demand in that interval exceeds L.

![[edf-process-demand.png]]

In this case, we could not use the $L_{BRH}$ since it requires to know all the Utilization and Deadline (in the formula). So only feasible way to use is $L_{LCM}$ which equals to 20.

In this interval $L=20$, we have these control points (L):
- With $\tau_1$ :  5 , 10, 15, 20
- With $\tau_2$: 6, 16
- $\tau_3$ is a little unique, since we have not known the exact value of $D_3$, but we know that it equals to $2C_3$ and also $5 \le C_3 \le 7$, then we have the control point of $2C_3$ (only one).

Summary, the array of control points for this task set is: ${5,6,10,2C_3,15,16,20}$

Remember the formula of $N_i^L.C_i$ we are going to calculate each value for each task (in each L, from 5, 6, ...)

Here, we have unidentified value of $C_3$, the simplest way is backtrack, try with each value from 5 to 7 (5,6,7).

![[Pasted image 20260313194437.png]]

In this image, we assumed (at earlier points) that $C_3$ cannot be less than 5 (due to the constraint) then we continued to calculate.

When we encountered a case that processor-demand could not be met, we changed the value of $c_3$ to another (5 to 6, but with that, we have a new control point, and must calculate it too).

![[Pasted image 20260313195336.png]]

Similarly, we calculated the processor demand at the new control point, when it was not sufficient, increased the value of $c_3$ again (which led to another new control point), then repeated these steps again.

But at last, at the very end point, we had an insufficient result, which we can proved that it could not be satisfy with any value of $c_3$ as the problem statement constraint. So that we did not need to calculate the last point (20).