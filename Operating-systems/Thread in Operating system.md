#OS #Foundation
## Appendix

### Thread structure
A thread in C programming language is implemented as a struct:
```C
struct thread
{
 /* Owned by thread.c. */
 tid_t tid;                          /* Thread identifier. */
 enum thread_status status;          /* Thread state. */
 char name[16];                      /* Name (for debugging purposes). */
 uint8_t *stack;                     /* Saved stack pointer. */
 int priority;                       /* Priority. */
 struct list_elem allelem;           /* List element for all threads list. */
 /* Shared between thread.c and synch.c. */
 struct list_elem elem;              /* List element. */
 **int64_t sleep_ticks;                /* time to sleep in ticks */**
 #ifdef USERPROG
 /* Owned by userprog/process.c. */
 uint32_t *pagedir;                  /* Page directory. */
 #endif
 /* Owned by thread.c. */
 unsigned magic;                     /* Detects stack overflow. */
};
```

- The `sleep_ticks` is a user-defined field to solve the busy-waiting problem, it is not mandatory in the structure declaration.
- The `thread_status` is a collection of pre-defined values: 
	- `THREAD_RUNNING`: indicates that the current thread are running at a given time. The function `thread_current()` returns the running thread.
	- `THREAD_READY`: this thread is ready to run, but it is not running now. It will be placed in the a double linked list called `ready_list` to be invoked next time the `scheduler` run.
	- `THREAD_BLOCKED`: it is currently waiting for something, maybe I/O or maybe an interrupt. It won't be scheduled again util it changes to `THREAD_READY` state. Can be achieved by calling `thread_unlock()`.
	- `THREAD_DYING`: had existed and will be destroyed by the `scheduler` after switching to the next thread.
- `allelem`: is a list of threads. It is used to link this thread  to all threads. Each thread is inserted into this list when it is created and of course, removed when it exits. The function `thread_foreach()` can be used to iterate through this list.
- `elem`: is a list of either ready or waiting thread. 

These `allelem` and `elem` are pretty confused at first but the first one is to link all the threads together to create a thread list (for the scheduler to track which threads are ready,...). The second one is for  putting this thread into waiting or sleeping list. They are both accept a list as input, but the purposes are different.