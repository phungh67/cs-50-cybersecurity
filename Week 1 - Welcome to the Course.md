#Introduction

# A. Cybersecurity meaning

>[!info] A real case
Nowadays, some devices only support a passcode of 4-digits. Each digit only accepts value in range 0 to 9. As a result, the number of possibilities are 10000, and with modern computers, it takes only milliseconds. But it will be another scenario with the help of ASCII characters, both upper case and lower case. In addition to these choices, punctuation is allowed to create a stronger password. The baseline for today's standard is "8 characters with at least 1 from these categories: upper, lower case, number and special characters".

# First definition
>[!info]
>Cybersecurity, also known as ==digital security or IT security==, refers to the measures and practices used to protect computer systems, networks, devices, and data from unauthorized access, damage, or theft. It encompasses a broad range of technologies, processes, and policies designed to ensure the confidentiality, integrity, and availability of information in the digital realm
## Questions - the balance between usability and security

> [!example]
> These devices are proved to be more expensive to have another one. With individual customer, it is a waste of money. Enterprise customers which consider the availability cannot afford a mass amount of fortune to equip these devices to all employees.

>[!example] Why 4-digits still exists?
> Some applications only required a passkey of 4 digits. The answer is pretty simple: the balance between reusable, convenient over strict security compliance. Of course the logic layer must ensure the basic authorization and authentication steps, but these passkey had been simplified to reduce complexity of "secret management" during development process.

## NIST - National Institute of Standards and Technology's recommendations about password compliance

>[!tips]
>The users should be left freely to choose which password that they are comfort with. But it must in the range of 8 to 64 characters since the longer password is, the harder to remember it.
>Another consideration is related to [[Attacks]], "verifiers" shall compare the prospective secrets against a list that contains values known to be commonly-used, expected:
>- [[Passwords]] obtained from previous breaches.
>- Dictionary words.
>- Repetitive or sequential characters.
>- Context-specific words, such as the name of the service, the username,...

>[!info] Even more security methods
>[[Authentication]] and [[Authorization]] are two phases that deeply involve in Security Assessment. They will provide additional layer along with **password** to protect user's account.



