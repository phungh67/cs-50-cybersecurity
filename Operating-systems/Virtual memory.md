Demand paging
- Only brings a page into memory if needed (if the page is requested by a process)
- Need new MMU functionality to implement demand paging.
- If pages needed are already in the memory
	- Nothing big, it is the same with non-demanded paging.
- If pages are not in the memory
	- Need to detect and load it into memory
		- No changes in both process/program behavior.
		- No need for the developers to change the code

Valid-Invalid Bits
- Using to denote if a page is loaded into the memory or not.
- Initially, these bits are set to "i" (invalid) during the starting phase.
- If the program tried to access a page with invalid status, get a page fault.

Page fault
- If there is a reference to a page that not resides in the memory, it will trap the OS to create an interrupt called page fault.
- This interrupt will end up will the OS will load that page from the secondary memory into the main memory, reset the page table and restart the instruction (of the requested program, of course!).

Aspect of demand paging
- Pure demand paging: start with no pages in memory.
	- OS sets the instruction pointer to first instruction of process, page fault when does not find the page.
	- Repeat this for every other process page on the first access.
- Hardware support:
	- Page table with valid/invalid bit.
	- Secondary memory (swap device with swap space).
	- Instruction restart (when completed loading the page into main memory).

When using Virtual memory, if there are 4 free pages and the process is 5 pages length, does it can run?
- Can run: this is proved to be the most correct because the OS just care about it can run or not, not depends on the ability to run concurrent or parallel.
- Can not run
- Can run if concurrent execution
- Can run if parallel execution

Copy-on-write
- Allowing 2 or more process to share a same page table.
- The process which has the need to modify a page create a copy to write it. That's why it is called copy-on-write.
- There is one famous bug called DIRTY COW (Disk Copy-on-Write) bug.

Page and frame replacement algorithm
- There are two main parts: 
	- Frame-allocation algorithm: determines how many frames to give each process.
	- Page-replacement algorithm: decides which page to replace. Want lowest page-fault rate on both first access and re-access.
- Page placement is to prevent over-allocation of memory by modifying page-fault service rountine to include page replacement.
- User modify bit to reduce overhead of page transfers - only modified pages are written to disk.

- FIFO: evicts the oldest page for new space. Affected by Belady's Anomaly.
- Optimal: evicts the page that will not be used for longest period of time, example for a string page: 70120304230321201701. The frame size is 3. In the time that 2 comes, 7 will be evicted because it has the longest unused period of time.
- Least Recently Used (LRU) algorithm: uses past knowledge (not future as the optimal), it replaces page that has not been used in the most amount of time. Associates time of last use with each page. Can implement a stack to ensure that the replaced page will be at the bottom of the stack.
- LRU approximation algorithms:
	- Needs special hardware and still slow.
	- Reference bit:
		- Associates a bit with each page.
		- If a page is referenced, set the bit to 1.
	- The reference bit is used to implement the second chance. If a page that will be evicted but has the bit is 1, it will be set to 0 and the algorithm will pick another page that has the same condition but the bit is already 0 and evict it.

Thrashing
- If a process does not have enough pages, the page-fault is relative high.
	- Page fault occurs when getting frame.
	- Replace existing frame of other processes.
	- But the evicted ones should be quickly put back if in needs.
	- This leads to:
		- Low CPU utilization.
		- OS thinks it needs to increase the degree of multiprogramming.
		- Another processes will be added to the system.
- Thrashing is a process that is busy swapping pages in and out.