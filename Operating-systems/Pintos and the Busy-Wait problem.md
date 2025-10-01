## Pre-define
>[! What is Pintos]
>This is an open-source UNIX like Operating System built for education purposes. So you can image it is a barebone OS, with some basic features but still has many rooms for improvements.

- This operating system has a basic Scheduler and know how to "effective" arrange processes.
- But since it has one of the most trouble some "busy-waiting", something must be done to address this problem

### 1. The problem
```C
void timer_sleep (int64_t ticks){
	int64_t start = timer_ticks();
	while(timer_elapsed(start) < ticks)
		thread_yield();
}
```
- `thread_yield()`: yield the CPU and insert the thread to `ready_list` (the thread is willing to give up it execution for other threads).
- `timer_ticks()`: return the value of the current tick (quantum time of the CPU).
- `time_elapsed()`: return how many ticks have passed since the `start`.

Look at the function above, we can easily notice that  the `sleeper` only does 1 thing: keeps checking the time till it passed the desired one. In this time, the CPU cycles are wasted and it not only thing to be waste, the power will be nothing as well.

First, we go to the basic flow-work for the waiting list and the sleep-wakeup method. The thread will have 4 main status:
- Ready (it was created successfully by the program, the OS,...).
- Running (executes instructions).
- Blocked (waits for some I/O or some instructions are blocked).
- *In addition*: Yield - the thread voluntary gives up its slot for other threads (preemptive).
- Dying (completes its function and wait to be completely deleted).
When the program calls the `thread_yield(&thread)`, a `thread` will be taken from the running list, modified (its property, to reflect its actual state) and will be pushed back to the end of the `ready_list`.

To have some inspirations for this problem, we will look into the `thread_yield` function:
```C
void thread_yield(void){
	struct thread* cur = thread_current();
	enum intr_level old_level;
	
	old_level = intr_disable();
	if (cur != idle_thread){
		list_push_back (&ready_list, &cur->elem);
	}
	cur->status = THREAD_READY;
	schedule();
	intr_set_level(old_level);
}
```

This function does a simple job: check a thread with `thread_current()`, if the current thread is not idle, it will be pushed back into the ready list. Remember to update the status of the thread and `schedule()` again.
The most important step in this function is disabling the timer interrupt with the block `old_level` and `intr_disable`. These instructions can be found on the official documentation about the Pintos in the Internet. We will dig in deeper about the reason of doing so.
The `timer` is the most important *interrupt* in the computer system. Because we know that for each core of the CPU, on job can be done at a instance point of time. The thing is, if the job is executed from the beginning to the end, many other processes will be starved. Each tick, the `timer` will wake the OS, the OS will look in the queue for the CPU and said "Hey, switch to this" (if some processes are already running enough). In the code above, we initialized the `schedule()`, which equals to a kind of a interrupt (to have the OS re-schedule threads). If a `timer` interrupt happens right in this time, the program can be stuck in the half-way-in-half-way-out. So that, to be safe, we should disable the interrupt before making any change for this.

### 2. Several inspirations
- From KAIST EE - Youjip Won:
	- Use `BLOCKED` status state for new `timer_sleep`. It will treat the voluntary thread as `BLOCKED`, the CPU will have time for other jobs.
	- Use `Sleep-Wakeup` alarm clock. We will maintain two lists: one for ready and one for sleep. When a thread is called by `yield`, it changes from the ready list (head) to the tail of the sleep list. And when the function `wake_up()` is called, the head of sleep list will be transferred to the tail of ready list (by `push_back`).

We will use the inspiration as above for the easy of contact.
#### 2.1. Alarm clock
Need to define a queue for sleeping threads.
```C
static struct list sleep_list;
```
There are two things to consider:
- Initialize it (How and when).
- How to put the list in the code (which file and which section).
#### 2.2. Global ticks and local ticks
Because we are going to disable the default timer, we should consider for a custom timer interrupt handle to check which threads to wake up.
With local ticks:
- Each thread should maintain its own ticks.
- Should go to the `thread.h`
With global variable:
- The minimum value of local `tick` of the threads.
- Save the time to scan the list.
- Should be initialized properly.
#### 2.3. Real implement
```C
// Inside the thread.h, make modification for the thread 
struct thread {
	int64 tick_to_wake_up; // tick till wake up time
}
// Inside the sleeper timer, check the tick each time it is called
void timer_sleep(int64_t tick){
	int64_t start = timer_ticks();
	if (time_elapsed(start) < ticks){
		thread_sleep(start + ticks);
	}
}
void thread_sleep(int64_t ticks){
	/* if the current thread is not idle thread
		change the stat of the caller thread to BLOCKED,
		store the local tick to wake up (defined in thread struct)
		update the global tick if necessary,
		call the schedule() */
	// should disable the interrupt while working with thread list.
}
static void timer_interrupt(struct intr_frame *args UNUSED){
	ticks++; //glocal tick
	thread_tick();
	/* @TODO
		check the sleep list for threads, if they are ready to wake
		move them into ready list and update the global tick
	*/
}
```

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

## Timer.c handling
```C
/* Sleeps for approximately TICKS timer ticks.  Interrupts must
be turned on. */
void timer_sleep (int64_t ticks) // this is a program-interrupt handler
{
 ASSERT (intr_get_level () == INTR_ON);
 // assign requested sleep time to current thread
 thread_current()->sleep_ticks = ticks;
 // disable interrupts to allow thread blocking
 enum intr_level old_level = intr_disable();
 // block current thread
 thread_block();
 /* set old interrupt level which was used before the current thread was blocked
 to ensure that no other logic crashes */
 intr_set_level(old_level);
}

/* Timer interrupt handler. */
static void timer_interrupt (struct intr_frame *args UNUSED)
{ // hence, it must have its own interrupt lol
 ticks++;
 thread_tick ();
 // Check each thread with wake_threads() after each tick. It is assumed that
 // interrupts are disabled  because timer_interrupt() is an interrupt handler.
 thread_foreach(wake_threads, 0);
}

/**
* Function for waking up a sleeping thread. It checks
* whether a thread is being blocked. If TRUE, then
* check whether the thread's sleep_ticks has reached 0 or not
* by decrementing it on each conditional statement.
* If the thread's sleep_ticks has reached 0, then unblock the
* sleeping thread.
*/
static void wake_threads(struct thread *t, void *aux){
 if(t->status == THREAD_BLOCKED){
  if(t->sleep_ticks > 0){
   t->sleep_ticks--;
   if(t->sleep_ticks == 0)
    thread_unblock(t);
  }
 }
}
```
