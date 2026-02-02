
A username is internally represented by a user identifier, or UID
- Special username are used for system functions, such as root, guest or apache (for all Apached-based application like HTTPd, tomcat,...)
- UNIX id command show UID: for example, when you logged in as "user1", the terminal will show something like `user1@hostname >`
- UIDs are stored in file `/etc/passwd` with username and preferred shell (`bash`, `zsh`,...)
- User's privileges are depended on the UID.
- Group id or GID is used to identify the a group of users. These users have the same permissions (and maybe role) within the system.
	- UNIX `groups <username>` shows the groups that `username` belongs to


- `/etc/passwd` file entry for user root:
	- For example: `root: AaecryptedPw: 0 :0 root : /root:/bin/bash`
	- As seen in the example above, we know that the `root` user has password `AaencryptedPw` with both user id and group id are 0. The root's home directory is at `/root` and the preferred shell is `/bin/bash`.
	- That revealed a lot of information, from the password (in plain text form) and the id, home directory,...
	- This vulnerability has been fixed by separating the user information and the password into 2 files, and only storing hashed password for comparison, not plain text one.
- The **root** user is the most powerful user in the system. As root, you can change password or information of all other users, as well as force them to login, log out, shut down the computer,... With root privileges, almost security restrictions are bypassed.
- Take over the root user is the most ideal goal for every attacker, since they can do basically everything with these privileges.
- Another user can assume the root role (gains desired permissions) by using the `sudo` command util log out. But luckily, this action always requires a password, except that the user was specified in the `sudoers` file, allowing this user to escalate its privileges without a password method (which brings a lot of security risks).

