Principles of this function:
- Intrusion
- Transfer of main program
- Settling down and establishing
- Continued intrusions

The worm first tries to obtain host addresses by examining files (system files), this is called "finding trust relations". It gets the information about trusted remote addresses, with these machines, the worm can propagate without additional authenticated actions.  The worm then tries to allow the daemon to execute a command interpreter and download code across a mail connection (with the aid of buffer overflows)