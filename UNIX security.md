
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


# 1. Password - Attack

Consider about the attacking model: surfaces, how to get access to password and the adversary model?

About the model:
- Insider: someone within the system
- Outsider: people from other places try to get into the system.
- Keylogger, eavesdrop to get important information.
- Privileged user gone wrong?
- Malicious root
- Social engineering attack (peak your screen?)

There are several methods that became standards for login-password desgined:
- Combination of multiple characters from different categories (lower case, upper case, special, numerical).
- Separation user information and password record.
- Never stored the plain text for sensitive information (password?).
- Encrypted maybe a good way, but how to enhance in the dictionary attack. Human brain is not designed to remember everything and be able to sort these things systematically? So it is necessary to protect the passwords that have same plain text.
- Salted bit is a method that randomly put a string, a character into the password of the user, then hashed it. And when checking, it only check if that string (input) can produced such a result. The old way is to hash it then compare.
- Keep important file in a secure manner - only specific person can view - which we call them as Access Control List (ACLs).

