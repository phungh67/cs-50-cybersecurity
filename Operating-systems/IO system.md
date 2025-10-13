The operating system is responsible for executing and processing the I/O operations, devices (that will include manage and control).

## I/O Hardware

There are several types of devices that can be plugged in into a computer system:
- Storage devices
- Human interactive devices
- Network devices.
Common concepts:
- Port: the connection point for these devices
- Bus: a set of wires and a protocol for data transmission between OS and these devices
- Devices can form a chain.

How can the processor give commands and data to a controller to accomplished a I/O transfer?
	- Processor read and writes a bit patterns in the controller register's
		- Use the special I/O instructions that specify the transfer of a byte or word to an I/O port address.
		- I/O instruction triggers bus lines to select the proper device and move bits into or out to the device register.
	- Device-control registers are mapped into the address space of the processor.
		- CPU executes I/O requests using the standard data transfer instructions to read and write the device-control registers.
		- In this way, CPU does not have to go through all the long way and is faster in the CPU perspective, and also does not need a specific kind of any instructions

I/O port typically consists of:
- Data in register: read by the host to get input
- Data out register: written by the host to send output.
- Status register
- Control register

Interaction between host and controller
- Busy-waiting or polling: the host is busy with trying to get access to the devices via controller.
- Interrupt: the controller alerts the host whether a device is ready to read
	- CPU hardware has a wire called interrupt request line
	- CPU senses the interrupt request line after executing every instruction.
	- If yes, the CPU performs a save state and jumps to the interrupt handler routine at a fixed address in the memory.
	- There is a need to have a sophisticated mechanism for this interrupt (power and time saving).
		- Non-maskable interrupt: reserved for events such as unrecoverable memory errors.
		- Maskable interrupt: can be turned it off by the CPU before the execution of critical instruction requests. (eg: timer in the[[Busy-waiting]], [[Pintos and the Busy-Wait problem]])
		- At the boot time, probes the buses to see which devices are present and installs the corresponding interrupt handler.
		- During I/O, devices raises interrupts when they are ready for service. These interrupts may mean:
			- output has completed.
			- input data are available.
			- or a failure has been detected.

Application with I/O interface
Blocking and non-blocking I/O
- Blocking
	- suspended till the I/O is completed.
	- moved from run queue to wait queue.
	- later moved back to run queue and resumes
	- easy to use and understand
	- insufficient for some needs.
- Asynchronous
	- Processes run while I/O executes
	- Return immediately without having to wait for I/O
	- Difficult to use
	- I/O subsystem signals process when I/O completed

I/O Scheduling
Device status table: when an OS supports async I/O and might schedule requests in different orders, it must be able to keep track of many I/O requests.

Buffering: using the storage space in main memory or on disk via buffering, caching, spooling