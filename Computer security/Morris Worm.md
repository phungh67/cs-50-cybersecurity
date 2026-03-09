Principles of this function:
- Intrusion
- Transfer of main program
- Settling down and establishing
- Continued intrusions

The worm first tries to obtain host addresses by examining files (system files), this is called "finding trust relations". It gets the information about trusted remote addresses, with these machines, the worm can propagate without additional authenticated actions.  The worm then tries to allow the daemon to execute a command interpreter and download code across a mail connection (with the aid of buffer overflows)

# 1. Sequence of the Morris Worm

First step is to conduct an intrusion, then transfer to main program, settling down and establishing (cracking account, hiding itself,...) then continue the intrusion.

## 1.1. The intrusion

Finding the cross-relation
Guest and crack the password
Use the debug facilities to sending mail handler
Exploit bug in a "buffer overflow" fashion.

So begin with a small piece, then after successfully cracked the system, the system will request for more worm.

The dangerous comes from the `rsh` or the remote login service, especially in the organization environment, because with the same security policies, same set of users and same set of operating systems, the Morris Worm can be spread in a rapidly speed.

The worm tries to find the host file `/etc/hosts` or equivalent file to **Finding the trust relation**, which machines can be logged in from this (infected) machine without any additionally authentication steps?

## 1.2. Guess the password

At that time, Morris thought that it was commonly for all systems (all machines in the same system to have the same password - who bother to think about complex password?), so that the worm tried to crack the local password with the unintentionally aid of the local dictionaries.

## 1.3. Hide and re-produce

There is a "trap door" at that time in the `sendmail` function of the SMTP mail server. More precisely, it was a bug in the debugging code, allowing the daemon to execute a command interpreter and download code across a mail connection (well, that is why each protocol must have its own port and restricted to do only a single purpose!!!).

Then we have buffer overflow.

