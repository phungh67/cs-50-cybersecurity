>[!Info] What will be covered in the Computer Architecture lectures?
>**Computer Architecture** - explain what is it and the role of "architecture" concept in the computer.
>**System organization and trends** - how to organize components and explain the trends of current design (more transistors or use these transistors more effectively).
>**Parallelism** - the differences between the parallelism in the threads level and the instruction level.
>**Quantitative evaluation** - how to measure in term of execution time, speedup time, arithmetic or geometric means and Amdahl's law.


# A. Concepts and vocabulary

`ISA` - instruction set architecture

# B. Communication and data-flow inside a complex computer system.

Normally, for single core system, all the components: I/O devices, memory (both primary and secondary) and the processor communicate via internal bus - a system bus - acts like a highway or a shared way so data can be transferred in it, so every components can exchange information, take the processed data,...

But in the high-end computer system (that is, a system with multiple processors running parallel), the communication between them may prove a challenge for designers and computer science scholars. Typically, the processor will come with a set of its dedicated memory and cached. Maybe the dedicated memory and cached are not separately designed and single-paired, but probably from a huge shared memory but organized in a hierarchy. And lastly, the interconnection bus between these processor+memory nodes.

>[!Conclusion]
>So at scale, the question is not about how to use, how to put as many components as possible into an area of computer (a PCB, a mainboard) but how to efficiency ensuring the internal communication between them. And also, since we are introduced with the concept of memory hierarchy - memory organization is considered as a top-priority.


# C. Parallelism in the computer execution

It depends on the task, for example:
- A scalar task should be accelerated by the instruction level parallelism.
- Otherwise, the thread-level parallelism can be applied. Since many threads from a program, a task can be distributed to different cores to run and to achieve near-real-time parallelism.

On the other hand, the parallelism could be done with data. There are two common ways: vector and array approach.
- With vector, the pipeline handles task concurrently, so for x elements, it will take x cycles to compute.
- But with array, x elements can be distributed to y array (depends on the memory organization), thus, can be completed in few cycles (fewer than the number of the element).


# D. Performance metrics and evaluation

So, we knew about architecture concepts, parallelism, but to judge which method is more efficient, several metrics should be taken on account.

The evaluation should be done with these concepts:
- Measure: execution time and throughput are the best way to tell if a method, a design is good enough or not.
- Compare: even the result is good, but without comparison, it is just a number, quite useless.
- Represent: benchmark and benchmark suite decide which workloads are used (and should be suitable with the system too).
- Summarize: means and reporting methods combine results across many programs.

## 1. Metrics

We already have the concepts, how about metrics?
- The first one is always execution time, response time or latency.
- The second one is the throughput - how many tasks can be completed in a period of time (or how big the data volume, which is successfully done).
- But the optimization could lead to different design choices, depends on the priority: latency or throughput, because we hardly achieve both at the same time.

## 2. Test suite

With metrics, we only have the first building block, because to achieve a meaningful result, we should carry the correct test suite. We do have many way to test a system, but to achieve a good result, it really depends on which type of test scenario we chose.
- Real program - the ultimate case, mostly used in the real time, but very difficult to port or to interpret the result (because we test on a system, maybe a bare metal without many abstract layers).
- Kernels - the very foundation case, every systems have it, but only narrow the scope into one or two functions (because you know, who cares about kernels beside programmers and nerds)?
- Toy benchmarks - simple and very easy to implement, however it only serves as a dummy.
- Synthetic benchmarks - controlled and totally predictable, but because of its artificial property, maybe it will not cover all the cases we need.

In summary, the benchmark suite will determine, or precisely, shape the final blueprint of our system.

With test case (test scenario) we also have test suite (a framework, a set of standards) we used to evaluate the result:
- SPEC - Standard Performance Evaluate Corporation suites for scientific, engineering and general-purpose workloads.
- TPC - Benchmark for transaction processing.
- EEMBC - Embedded benchmark suites.
- Media benchmark - Workload specially designed at media processing.

## 3. Reporting performance 

>[!Info]
>In short: the numerical methods and a bunch of formulas to follow.

Weighted arithmetic mean:
$$
\sum \frac{T_i}{N} = \sum T_i*W_i 
$$
With $T_i$ is the execution time of program $i$, and $N$ is the total number of evaluated programs, or in weighted scenario, the $W_i$ indicates the weight of program $i$

Speedup over a reference machine $R$
$$
S_i = \frac{T_{R,i}}{T_i}
$$
With the numerator is the execution time of task $i$ in the machine $R$ while the denominator is the execution time of task $i$ in the current machine

Lastly, the geometric mean (~related to geometry, so we have the root guys)
$$
\bar{S} = \sqrt[N]{{\prod_{i=1}}^N S_i}
$$
Literally the production of speedup time for task $i$ then we use the root $N$ - with $N$ is the number of the task.


>[!Note]
>As we see, the geometry maybe the most reliable way to evaluate, but since sometime, we just need the mean, not the speedup. However, the mean can be misleading, because for example, a very long task can cause the mean shifted a little bit longer, so, maybe not good all.

# E. Amdahl's law

>[!The law]
>Overall speedup is capped by the part of the execution that cannot be accelerated.

For example, a task $T$ has two parts: $F$ - the fraction that can be accelerated, so we have the capped as follow, with $S$ is the factor of enhanced for the part $F$
$$
T_{exe}(withE) = T_{exe}(withoutE)[(1-F)+\frac{F}{S}]
$$
And with that, we have the speedup:
$$
Speedup(E) = \frac{T_{exe}(withoutE)}{T_{exe}(withE)} = \frac{1}{(1-F)+\frac{F}{S}}
$$
