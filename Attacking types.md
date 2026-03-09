
Let's start with the definition about "attacking", how can we define what can be called as an attacking scenario?
In term of "computer security", an **attack** is some action that targeted a system (computer system) and exploited that system's weakness in order to take some advantages. Not all attacks must be fatal, it can be a small attack with electricity outage, a simple rounding bugs or something slowing down the system. The purposes of these actions are also varies. To embarrassed the system's owners/developers? Or simply to prove that the targeted system is not invincible as it looked to be.
Take an example of these code:
```C
#include <iostream>
#include <fstream>
#include <string>

int main() {
	ofstream myfile;
	string input_line;
	
	cin >> input_line;
	myfile.open(input_line)
	myflile << "Hello World!";
	myfile.close();
	return 0;
}
```
The code above can be harmless at first look, it just tried to read some input and open the corresponding file, of course, if the user just typed some garbage value, an error would be raised "cannot open file/ the directory is not existed,...". But this file is extremely dangerous when the attacker has knowledge about SUID.
The SUID is a very important bit in Linux/Unix file permission (check [[Filesystem|file system overview]]). That is, for every executable files, there are 2 things must be understand: SUID and EUID.
First, SUID indicated the Set-User-ID, which means, when this bit is on, that running file would be executed with its owner's permission rather than caller's permission. Example: when file *a.sh* is owner by root, and the executor is just a normal user, in that scenario, this process would run with root privileges. On the other hand, EUID is the effective UID, which indicates the name of user the process runs with. That's the reason why SUID is an extremely dangerous property in Linux/Unix file system.
Turn back to the above code, if that script is run with SUID but (and appropriate right), it will overwrite the content inside the file specified in `input_line`, for example, a password file, and in that case, many users (all) will lost the access to the system.
So we have the first definition
>[!Malicious Code]
>This term is used to call any piece of code which is added, changed or removed from a software system in order to intentionally cause harm or subvert the intended function of the system. It is still normal code, it is not written in some form of "alien" language or something that we cannot understand, it just aim to corrupt the system. Another term for this is "**Malware**".


The trend of the **Malicious code** is also evolved recently due to the rapidly improvement of the computer system.
- "Growing number and connectivity of computers", so everyone is connected and dependant on computers. This mean attacks can be launched from everywhere and easily (automatically and remotely).
- "Growing" system complexity means more vulnerabilities to exploit, more places to attack and easy to hid the malicious code.
- "Systems are easily extensible", because there is a massive numbers of loadable modules, easily used to attack.

Even the dangerous ones have their own "categories". There are several types of malicious code which can be classified based on behavior and blast area:
- Traditional virus: the earliest type of malfunction code, this is usually attached to existing program code, intervenes in normal execution then replicates and propagates.
- Document virus: they hide inside highly formatted documents included commands (excel).
- Stealth virus: hides the modification it made to the system, normally by monitoring system calls and forging the results of such calls.
- Polymorphic virus: avoids virus scanners by producing multiple variants of itself or encrypting itself.
- Hoax is not really a virus but more a warning email.
- Rabbit: multiple fast to exhaust the system's resource.
- Worm: standalone program that replicates and spreads copies of itself over the network. The most dangerous property of it is non-trivial to make.
- Trojan horse: disguises itself as a normal program but have some hidden functions inside it, and of course, these features are unwanted by the users.
- Logic bomb: triggered when a condition (or several conditions) is met.
- Time bomb: denoted by a time condition.
- Trap door: left unintentionally during development and system design phases. These doors were used for testing purposes, but forgotten after release, making a potential threat.
- Salami attack: achieving some economic benefit but making a large number of insignificant changes (rounding error,...).

[[Love Letter Attack]]
[[Morris Worm]]
