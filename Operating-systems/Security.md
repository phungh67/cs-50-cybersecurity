The operating system and processes that are belong to the OS are these "common things". But there is a problem that, when OS executing commands or instructions from user, the risk of leaking information between dataflow is vital.
These "attackers" may want to read, modify or deleting somethings in the system. They can take advantages by vulnerabilities of these software to overrides or overwrites to file system.

Attack types:
- Worm: freestanding program that propagates without user interactions.
- Virus: attaches with other program, propagates with user interactions.
- Trojan: malware disguised as legitimate software.
- Dos, DDoS,...

Classified of attack according to dimensions:
- Passive: sniff network traffic and try to break encryption.
- Active: take control of a browser and use it to execute code.
Protection mechanisms:
- Cryptography.
- Software hardening - best practice.

## Protected domains
- Domain: set of <object, right> pairs.
- A domain refer to 1 user but can be more general than that (a group).
- Security works best when each domain has the minimum objects privileges to do its work (Least authority).

The most common is **Access Control List**. 
**Capabilities**: each object can be contained in a capability list could be a pointer to another capabilities list (to facilitate the sharing subdomains).
- Tagged architecture: each word in memory has a special bit that can be only changed in the kernel mode, containing a capabilities.
- Keep C-list in OS memory, user process can point to them.
- In user space, these things are encrypted so the user can not make any modifications.

## Authentication
The password problem:
- Never store the password and user information in a linked object without protection.
- The most primitive is to store only username and the encryption string that represents the actual password. This can be attacked by the brute force, try all the common passwords or the combinations till found.

### One time hash function  - for the one time password
- Use hash function f with input and output having the same length.
- Chose a number `n`, to represent how many passwords.
- P1 will be result of hash `n` times from a source. P1 also is the hashed of P2. But cannot find P2 from P1.
## Exploiting software
- Cracking passwords is only one way of course to gain control.
- There are several ways:
	- Drive-by-download: infect user with illegally software.
	- Exploit bugs of the OS.