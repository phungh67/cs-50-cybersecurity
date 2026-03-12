
This is the detail about the first feasibility test conducted in a system to determine if a task set can be scheduled or is there any existing schedule algorithm for that set [[Scheduling in modern system]]

# 1. How long to inspect (what is the Hyper Period?)

For a task set that is executed *synchronously*, that $\forall i,j$ : $O_i = O_j$ (with O = offset), it is sufficient to investigate the interval [0, P] where P is the hyper period of the task set.

In contrast, with *asynchronously*, because the offset of each task is not the same, only sufficient to take P if there is no task arrives after P, but otherwise, more than one hyper period is necessary.

# 2. Cyclic executives

From the motivation of **HPA**, use the table-based schedule. There are three properties:
- Table-based schedule.
- Feasibility test performed when generated table.
- Schedule repeats itself (= cyclic executives).
- Off-line schedule generation.
- Mutual exclusion is handled explicitly.
- Precedence constraints are handled explicitly.

The length to inspect (a.k.a HPA) can be generate by using LCM (least-common-multiplier) of all tasks.

![[HPA-sample.png]]

In this example, we have 4 tasks: $\tau_1$, $\tau_2$, $\tau_3$ and $\tau_4$ , with the offset as the table (the arrival time of tasks). Also, the $\tau_3$ can only run if both $\tau_1$ and $\tau_2$ were completed before.

![[HPA-guide.png]]


With the EDF - Earliest Deadline First, which task has the closet deadline will win. Also in this case, task is allowed to preempt each other, so that we have the above diagram.

First, $\tau_1$ arrived with offset = 0 and deadline 7, which won because $\tau_2$ also arrived at 0 but had deadline of 12, also, $\tau_4$ deadline was 1 + offiset (which was 5, so it would be executed right after $\tau_1$, but it only arrived at 3).

But in the time of 8, the system's state changed, because $\tau_4$ arrived. At that time, the deadline of $\tau_4$ was 4 while $\tau_1$ was 7, so the $\tau_4$ won, $\tau_1$ was preempted. Similarity, we have $\tau_2$ won after $\tau_1$ and only after both task, we were able to execute $\tau_3$ with some preempt since the deadline of $\tau_4$ won (since it only takes 1s of deadline and 1s of execution).

Also, with the EDF to test the feasibility of scheduling, on the formula is not enough (take some formula from Liu and Layland's in [[Pseudo-parallel execution]]) (but take the right hand side to one). If all tasks have the constrained deadline (less than period) we must go to this [[Feasibility-test-single-processor]]
