
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
![[Pasted image 20260312220659.png]]