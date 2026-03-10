
With various ways to attack (mentioned in [[Attacks]] and [[Attacking types]]) and with the consequences from crash and kill to the exhaustive the system [[Denial of Service attack]], we want to not only build a system that can mitigate these threats, but also want to build some early warning, alerting stack to detect the abnormal activities.

This is only the burglar alarm, cannot react in a defensive fashion to chase of the intruders. "Prevent the threats is better than fighting with the threats".

# 1. Motivation

Even if unsuccessfully blocked the intrusion, but the value of this detection comes in the form of "important" feedback to continuously develop and harden the system. Main purpose of the Intrusion Detection Service (IDS):
- Detect intrusions and intrusion attempts.
- Give alarms.
- Stop on-going attack (but not quite in the term of "cut-off-immediately", in the term of response).
- Trace attackers.
- Investigate and assess the damages.
- Gather information for recovery actions.

# 2. Details

In fact, login is a mean of intrusion detection, it tries to detect if any attacker tried to use another user's credential to log in? System call also tries to investigate and put a stop to any program that acted suspiciously?

So what we try to detect?
- Sniffing the password string (try to many times?)
- Buffer overflow attack? Was user trying to put as many characters as possible every time?

>[!Components]
>Control, reference data, logging & data reduction, analysis & detection.

The control of course is the main "brain" of the system. The reference data acts like a single source of truth for this detection. Logging and data reduction aim to take input data, reduce them to only take meaningful part. Then the analysis and detection will recognize attack patterns from "reference data" and "actual data".
# 3. Limitation and requirements

False alarm - that is, a totally normal activity but due to some latency, something from human,...

An IDS system needs to satisfy:
- Response time, which is equally with the speed of the system.
- Fault tolerance, must withstand with failure of the sysem.
- Ease to integration, usability and maintainability.
- Portability.
- Support for reference data update (up-to-date).
- "excess" information (access no more no less - privacy).
- Cost (CPU, memory, delay,...)
- Host-based or Network-based
- Security of the IDS itself.

# 4. Practical problems

>[! False alarm]
>Many alarms within the system, and the trade-off of covering all the attacks with the number of notifications you will get. It is mandatory to investigate all the alarms if want to improve the accuracy of the IDS. But of course, these efforts are indeed time-consuming.

>[!Adaption] 
>To get a proper protection for system is impossible. IDS must be tweaked and trained to match with requirements and to fit with system's behavior.

>[!Scalability]
>Network-based speed, one sensor, many sensors and Internet of Things problem.

>[!Test method]
>Lack of test case and verification standards.

