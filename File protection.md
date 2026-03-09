In UNIX based system, everything is a file. File comes with information that let us know which is the creator of this file and who can interact with it (read, modify, delete,...). As we knew, in UNIX, even hardware is a file, so if someone with sufficient permissions, they even can modify the characteristics of the corresponding components, hence compromised it.

Classes and permissions in file protection:
- File permissions indicate ***who*** can do ***what*** on a specified object.
- Class - indicates the entities: owner, group and other. So the owner is who created this file, of course. Group refers to users in the file's group and other is the rest (except the super user).
- Access - indicates what actions can be done: read, write and execute.

Some file should be only made available for only its owner, including: private key, paraphrases, revoke certificate,... Otherwise, the attackers can exploit these holes to guess or break the encryption of sensitive files. 