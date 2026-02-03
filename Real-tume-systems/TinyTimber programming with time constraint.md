
There are three options for time constraint in TinyTimber kernel:
- AFTER
- BEFORE
- SEND

The SYNC() marco:
- SYNC(object, method, arguments)
- ASYNC(object, method, arguments)

The SYNC call blocks the calling thread or process and only return when the method inside it is completely executed (even error or success). The ASYNC on the other hand, returns immediately.

AFTER(baseline, object, method, arguments): executes a method within an object with supplement arguments after the baseline
BEFORE(deadline, object, method, arguments): same as AFTER but it is an celling bound (the other is floor bound).
SEND(baseline, deadline, object, method, arguments): ensure the action is within a window of time between (baseline; deadline).

When using BEFORE or AFTER, it is critical to imagine and plan carefully for which tasks (methods) should be complete within a time frame.

In addition, SEND can be used to call a delayed method.

Example for a periodic tasks:

*"Implement two periodic tasks with a shared object in C using the TinyTimber kernel:
We assume that:
- An object `actobj` type of `Actuator` is shared by two periodic tasks `task1` and `task2` with periods of 300us and 500us.
- Both tasks may concurrently call method `update` of shared object `actobj` with value 10 and 20 respectively.
- Adding deadline 100us and 200us for these 2.

---

