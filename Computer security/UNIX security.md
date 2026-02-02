
## Security ideas on UNIX
- Memory protected for processes:
	- Each processes have own address space.
	- Communication with hardware is done through the operating system.
- Files are protected between users:
	- Since everything is a file concept is universal within UNIX systems and various distros that follow this concept, all object should be applied a same protection method.
- Maintenance is carried out by a "reliable" system administrator:
	- Also known as root or super user.

There are several concepts to cover in this note:
- [[User and groups]]
- [[File protection]]
- 