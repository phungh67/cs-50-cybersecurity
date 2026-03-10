
In the security terms, we usually consider about the Confidentiality, Integrity. And the most feasible method to do so is forcing user to authenticate before entering the system. In an isolated environment (a single workstation, or a system that is not connected to the internet directly), we have [[UNIX security]]

# 1. Spoofing

Try to pretend that you are the person client tried to communicate with.

# 2. Kerberos

It is an authentication system between users and the servers. The original requirements for this method:
- Secure - avoid eavesdropping.
- Reliable - service must be available when needed.
- Transparent (to the user, at least they understand the result, granted or not).
- Scalable

The system based on a secure Kerberos installation, where account name and password are stored.
Other nodes in network maybe insecure.
Password is not stored locally, stored at Kerberos instead.
Password are never sent over the network.
The authentication gives a basis for authorizations.

There are several components of a Kerberos system:
- Ticket - a token used by the user, so that his/her identity can be securely transferred to the server. Inside it is the necessary information needed for the user and server to be able to communicate (example: cryptography key). One ticker per service, usually valid for hours.
- Authenticator - a one-time token showing that the user has the permission to use a service, short-lived, only use to verify the user's identity.
- Session key - a temporary key for the communication between the user and the service.
- Life time, time stamp and nonce are used for controlling the ticket, ensure that the ticket only valid for `timeStamp` + `lifeTime` and the nonce is a random number stamped to prevent relay attack.

The authenticate protocol:
- User put some credential, the Authentication server will work, compare the information that user provide or something (user only has to do once per logon). If succeeded, the authentication server will provide some kind of information for user.
- User then uses it to the ticket granting service to get the ticket.
- Use that ticket to access to the server.

The message that user sent to the authentication server: `ID`, `ID_ticketservice` and `TimeStamp`. These things are wrapped in plain text, since it literally does not contain any sensitive. But in return, the server returned a key (session key from the use password, only can be decrypted with correct password), all the things that user sent earlier, some information and the ticket. This ticket can only be decrypted by the key shared between the ticket service and authentication server, to make sure that user cannot change it.

But this ticket is only for requesting the actual ticket to service. The main ticket is encrypted with the session key (that is affected by the life time of the token) shared between client and ticket granting service, found in the ticket.

# 3. Vulnerabilities of Kerberos

Everything has failure. And of course, in every steps, no matter how secure it can be, there is always some things, some workaround methods to bypass, to exploit, to sneak into the system.

The TOCTOU flaw is referring to the bug in the source code of `Kerberos` version 4. The intention is quite simple:

When Kerberos check a file to open, the attacker somehow create a regular file, for example `tmp.txt` right at the directory that Kerberos was currently checking. When the code executed another instruction (must be done after it checked and verified that the file `tmp.txt` actually existed and was normal), attacker then deleted the actual file, create a sym-link to the target file. Because the code is only check if that file is valid or not (and of course, in the Linux operating system, everything is file), so the sensitive file can be open and even modify illegally.