
As in [[Firewall]] chapter and a small question in [[Exam 202508]], Intrusion Detection System is an important tool, used in conjunction with firewall and other protection mechanisms to achieve the system's security and reliability.

>[! Overview]
>An `IDS`is passive and sends alarms when rules are triggered. It typically sit between users and the systems (maybe after the firewall is the best setup).
>An Intrusion Prevention System `IPS` can take actions, for example, block packets and modify some firewall settings: `BLOCK * FROM 10.1.1.44 FOR 10 minutes`. But this system can sometime enable `DoS`attack (modify firewall mistakenly can lead to deny legitimate packets, hence making clients unable to communicate with servers).

## Types of Intrusion Detection System

**Network-based** - looks at the traffic on a network segment to detect suspicious.
**Host-based** - individual `IDS` located in individual hosts

## Centralized logging for better monitoring - management

Take the example from a diagram below

```Text
Central machine: log files __________________ Host-based IDS (HIDS)
|        |
|        |̣̣_____________ agents 
|        |̣̣̣̣̣̣̣_________________________________ Main border Firewall
|
Switch and Router
Network IDS (NIDS)
```

The logs from all types of components were transferred to a central processing unit. An intelligent logging with analysis solution could be installed in it to increase the efficiency of log aggregated and analyzed.

AI or Machine Learning (ML) are frequently applied to the log analysis. It was arranged into a three-dimension space: threat severity - asset value - confidence.

Also in the IDS, sensitivity is a compromise, too much - a bunch of false alarms, but too little, we are allowing true positive - true malicious payloads sneak into our system.

There are two formulas for this
$$ Detection Rate = \frac{True Positive}{True Positive - False Negative} $$
$$ Flase Alarm Rate = \frac {False Negative}{False Negative + True Negative}$$
## Signature and Anomaly Based Detection

### Signature Based - misuse

- Applicable for known attacks, known patterns, as long as the mechanism, patterns, signatures,... are known, this is the best strategy to deal with these stuff.
- Few false alarms.
- Cannot stop the new attacks (which have zero known patterns).
- Cannot detect zero-day attacks targeting vulnerabilities that are not yet known.
- Can be compared with an anti virus software.
- Free multi-platform tool owned by Cisco.

### Anomaly based
- Detects anomalous, unusual, behavior - especially new ones to the network and also can look at statistical patterns.
- Only way to stop zero day attacks.
- Needs training - bring a hard question: which is normal?
- Many false alarms.
- Hardly to measure what attacks and which patterns it has learned.
- Can be compared to anti spam software (and if it is similar with Google Mail, totally crap).

## Packet processing 
![[Pasted image 20260523152548.png]]

Example about a rule

```Bash
# syntax
aler <protocol> <scrIP> <srcPort> -> <dstIP> <dstPort> (options)
# examples
alert tcp !10.1.1.0/24 any -> 10.1.1.0/24 139 (msg: "External Netbios traffic";)
aler tcp any any -> any 80 (content: "GET/cgi-bin/", msg: "Attempt scan";)
aler tcp any any -> any any (flags: SF; msg: "SYN-FIN scan";)
```

Example about Cisco IOS Firewall

![[Pasted image 20260523152958.png]]

# Evading Firewall and Network IDS system

## Techniques

Many, but need to know the most efficient and importantly - left behind as less footprints as possible.

**Main problem**: Network IDS must know how end-systems interpret packets to take effective.
**Main categories of the problems**
- Insert - the Network IDS accepts the packets end-systems reject.
- Evade - the Network IDS discards the packets end-systems accept.
- Denial of Service - the Network IDS cannot keep up with the load.

Examples of packet that end-systems will reject:
- `IPv4`, `TCP`and `UDP`packets with checksum error (v6 does not support checksum).
- IP length different with link layer length.
- `TCP` options present that may or may not be accepted (system-dependable).
- Expiring Time-to-live.
- IP fragmentation (different assemble policies).
- Overlapping IP segments.

Otherwise, the analysis of application level protocols can be extremely complex (many prior layers, many information in it,...).

### Expiring Time to Live to evade IDS

Set different Time to Live to a single message but different parts (fragments). The time-out pieces can never reach the system, hence the IDS also was not able to detect these suspicious things. And on the receiver's side, the correct "malicious" phrase was assembled.

![[Pasted image 20260523154848.png]]

### IP fragment reassembly timeout 

Any system will have a maximum waiting time for all fragments to be reassembled, after that, drop.

In the case the IDS has shorter timeout than the system:
- Attacker sent frag-1.
- Then IDS timed out.
- Attacker sent frag-2.
- Host reassembled the data.
- The IDS timed out frag-2

So in this case, attacker still succeeded in send the malicious payload and evaded the IDS.

In the case that IDS has longer timeout than the system, still reassemble the malicious payload, but this time, maybe both IDS and system have the payload too.

![[Pasted image 20260523160216.png]]

## `TCP` session and IDS system

In the scenario of communication over three-way handshake, the IDS should be able to keep up with the nature of `TCP`connection to not unintentionally disrupt the service.

#### First scenario - must observe a valid handshake.

The IDS must witness a valid three-way handshake with `SYN-SYN/ACK-ACK`before allocating a state in its tracking table.
- A packet out of nowhere will be immediately dropped.
- Requires the sequence numbers to be match the initial handshake setup perfectly.

Maybe not good in a case of high-load network, causing some packets to be dropped - then cannot enter due to IDS must request strict three-way.

#### Synchronized on data.

Allowing inspection even without a success handshake. Hence, allow the IDS to involve with the packets were started before the IDS was booted up.

The disadvantage: The attackers can forge a fake packet to attack the system, because the IDS now allow the "out-of-nowhere" packet to enter. So the Ingress filtering from the firewall is needed.

And the most challenging question is how to handle the packet if a new handshake is seen. Choose the new, or choose the old one?

![[Pasted image 20260523163547.png]]