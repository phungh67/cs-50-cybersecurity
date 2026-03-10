
At first, we already have 2 background concepts: [[Cryptography]] (to protect information in-transit, at-rest) and the [[UNIX security]], the built-in protecting mechanism of the operating system itself. But we also have some active responses that we can do it at will

# 1. Forewords

Common assumptions for traditional programming:
- Inputs of the program.
- Environment that program will run in.
- A "cooperative" user within program context.

But in the **Defensive programming**, the code is always reused - so it is important to keep in mind that "NEVER TRUST THE USER INPUT".

For example:
- We should always validate the assumption, whether the input is correctly, or someone tried to make fun with malformed input?
- The environment that program will run in, is it safe, is it tolerance to failure, or maybe it is an out-dated operating system that contains bugs from many years ago?
- Also consider the techniques used by the attackers.

In real scenarios:
- The database is used by almost all real systems, so attacking database is a never-out-date movement.
- A web application can be made quickly by any junior programmers, but accessed by people all over the world, which means, many threats.

# 2. Domains

Handle program input to prevent two common things: [[Buffer overflow attack]] and [[SQL Injection]] . Keeping these in mind, we have "writing a safe program code". Then also need to consider about interaction between OS and other programs. Lastly, handle the program output correctly, to make sure that even the attackers somehow step into the vulnerabilities, they cannot tell they was success or not.

An example is, a web service sometimes can execute some shell commands to return the desired resources to user. But we know that, whatever command the web application executed, it would have the same permission as the application. And sometime, because of not correctly handling the user input, user can trick the program to execute malicious command, for example `print `\usr\bin\finger --sh $user ` ` for example, if the application was running as root, all new command can be run as root too.

# 3. Mitigation strategy

There are two approaches:
- Define what is valid, then block everything else.
- Define what is invalid, then block if it matched with input.

But the problem with case 2 is maybe you can miss some cases.