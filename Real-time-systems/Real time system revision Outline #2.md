
This is the outline for revision - resit exam #1 June 2026

In the concepts of real time system, the most important thing is know if a set of tasks can be scheduled in a given system (single core or multi cores). To do so, we have two conditions: the feasibility and the schedulability of a set.

Also, we have necessary and sufficient condition.

With a necessary condition, a no means no, but a yes can not ensure that is it possible to carry out the test.
With the sufficient condition, a yes means yes but no does not ensure it is impossible.

So to prove a task set that is possible to schedule (or unschedulable), need to perform the exact test (both necessary and sufficient property).

>[! Note]
>But before any feasibility test, the primary motivation is the worst time execution analysis by Shaw's. It is a way to estimate the execution time of any code block or program in the worst case (longest path of a code snippet).

This method is quite useful in some early generation of processor, in which the pipeline execution and the caching were not applied. With new methods and especially the RISC-V family, the `WCET`
 become more and more challenging.
 
 Check the [[Worst execution time exercises]] for more practical problems and how to solve them.
 Note that, for RISC-V processor, we have several visible challenging:
 - Pipeline, leading to source of time variations (data conflict, branch conflict and even cache missed) -> so that the cache itself is a source that affect tot the result of worst time analysis.
 - With the cache memory, the main challenge comes from different caching methods between data and instruction.
 - With the pipeline execution, the conflict can happen between code blocks.

## General way about scheduling
[[Scheduling in modern system]]

## The first and the most primitive way to analyze a task set

>[! Formula] 
>The Liu & Layland's feasibility test with simple formula

Check note in this [[Pseudo-parallel execution]] - to understand about the formula and in which case we can apply only it to prove that a set is safe.

## The second method - long and very confusing (if the period is big) but surely can handle the outcome

>[! Note]
>This is a diagram, need to keep in mind some principles: arrive is arrow down, complete is arrow up, arrive does not mean executed right away. Task can preempt each other if the deadline is shorter.

Check the note about [[Hyper Period Analysis (HPA)]] - instruction, example about it. Very easy to understand.

## When all of these above were failed, we move to more complicated methods

>[! Defintion]
>Response Time Analysis (RTA) we consider in a given deadline, if there is any iteration that has the same result, we say that task is convergence in permitted time. Hence, if all tasks are convergence, the set is feasible.

Check the note [[Pseudo-parallel execution]] section 3 for more information.

Depends on the priority of the task (rate monotonic or deadline monotonic), we decide the factor in each task's RTA formula. The interference factor is **rounded down**.