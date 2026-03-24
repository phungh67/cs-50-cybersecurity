Refer to the threat comes from tricking the program to execute some logic evaluation so fast that it optimistic assumed that the condition was unchanged (TOCTOU attack [[Attacks]]), this course aims to give two outcomes in parallel: **attack** and **defense**.

## Project topic

It's not mandatory but it's good to think about it now (since the deadline is 10 days):
- Set to open-ended (a pre-defined problem with desired outcome or problem with open closure).
- Common attacking types and protection, software vulnerability studies, OWASP top 10.
- Security tools: race detection tool,s obfuscation tools, OWASP topics,...
- Heartbleed, Spectre/Meltdown, Log4Shell, Ransomware,...
- Android app security.

# 1. General problem: malicious and/or buggy code is a thread:

Trends in software:
- 3-rd parties code (since we don't really want to reinvent the wheel, there are many snippets from paid to open-source).
- Platform independent - run in everywhere, so the risk of being contaminated is higher than normal.
- Extensibility.

Led to many opportunities for attackers:
- Easy to distribute exploits.
- Write an attack once, run everywhere.
- Systems are vulnerable to undesirable modifications.

# 2. Principles of Security Design:

## 2.1. Least privileges

>[! Definition]
>Each principal is given the minimum accesses needed to accomplish its task, no less, no more.

But on problem is **OS Coarse-grained control**
- Because the Operating System controls and enforces security policies at system call layer.
- So that, it is difficult to control the application when it is not making any these calls.
- Also, the security enforcement decisions made with regard to large-granularity operating system abstracts: files, pages, sockets, processes.
- Furthermore, Operating System keeps relative little state (recorded in files): recently communication from other machines (mail, messengers,...) or how many TCP connections have been made during last minute?
These problems raised a new need: **fine-grained control**
- Modern programs control security based on the application abstraction.
- Need extensible, reusable mechanism for enforcing security policies.
- Language-based security can support an extensible protected interface.

## 2.2. Keep the TCB small

>[! Definition]
>Trusted Computing Base - TCB, component whose failure compromises the security of a system. In other word, it contains all things in the operating system that acts as a source to enforce security policy on the machine. So that, small TCP, could not guarantee 100% protection, but can be checked, reasoned and tested easily, most likely to work. But the larger the TCP is, the more likely it will have bugs that could be compromised.

Defense against malicious code:
- Analyze the code and reject in case of potential harm.
- Rewrite the code before executing to avoid potential harm.
- Monitor the code and stop it before it does harm.
- Audit the code during execution and take policing action if it did harm.
