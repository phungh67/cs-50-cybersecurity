
# 1. IP table

The software is a part of a very large bundle calls `netfilter`, there are many modules, many tools were included in it, and `iptables`is a tool to manage the firewall rules for Linux system. It is a little complicated, so nowadays, people have another thing `ufw`- literally uncomplicated firewall, easier to use, but the old fashion way still gives many knowledge in deep, even close the the kernel system and some under the hood things.

To understand this better, let take a look at behind the scene concept about `netfilter` with document from [Introduction guide about net-package-filter](https://www.netfilter.org/documentation/HOWTO/packet-filtering-HOWTO-3.html)

And finally, the UI of this software `iptables`will be like this
```Bash
## Insert connection-tracking modules (not needed if built into kernel). 
# insmod ip_conntrack # insmod ip_conntrack_ftp  
## Create chain which blocks new connections, except if coming from inside. 
iptables -N block 
iptables -A block -m state --state ESTABLISHED,RELATED -j ACCEPT 
iptables -A block -m state --state NEW -i ! ppp0 -j ACCEPT 
iptables -A block -j DROP  
## Jump to that chain from INPUT and FORWARD chains.
iptables -A INPUT -j block 
iptables -A FORWARD -j block
```

In the above snippet, user tried to create a new chain named "block", which filtered all connections with state "ESTABLISHED, RELATED" will jump to except (all on going connection will be accepted). Also, all internal connection will be safe too. But otherwise, block them.

A simple illustration for this tool:
```text
                          _____
Incoming                 /     \         Outgoing
       -->[Routing ]--->|FORWARD|------->
          [Decision]     \_____/        ^
               |                        |
               v                       ____
              ___                     /    \
             /   \                  |OUTPUT|
            |INPUT|                  \____/
             \___/                      ^
               |                        |
                ----> Local Process ----
```

# 2. The lab

>[! Question 1]
>How and when is each chain is used?

Simply enough, there are three default chains: INPUT, OUTPUT and FORWARD.
- INPUT will be used for all incoming package that was specified that should be delivery to this machine (destination == this machine).
- OUTPUT will be used after the machine has done the processing packets, then wanted to send it to the destionation (source == this machine).
- FORWARD will be used when a packets came to this machine but the destination was somewhere else. In most case, DROP will be executed.
>[! Question 2]
>What is the effect of dropping echo-reply packets in the OUTPUT chain? Use the figure to illustrate the path of the packets. Mark the path with arrows and use and X to mark the point where the packets are dropped.

The packets are still received, then the machine still worked to produce some reply, but the reply will be dropped due to the rule. Machine still has to compute, still has to consume some resources to forge the response. But the response packets never came out, since it would be dropped once it hit the OUTPUT in FW.
>[!Question 3]
>What is the effect of dropping echo-reply packets in the INPUT chain? Use the figure to illustrate the path of the packets. Mark the path with arrows and use and X to mark the point where the packets are dropped.

Similar to above option, at the end, no replies will be sent back. But instead of dropping the packets after finishing prepare the response, the packets would be dropped immediately when it reached the machine. This ensure that machine will pay no attention at all, not wasted any resources.
In security, maybe it is better, at performance too. But in the case of the MTU path discovery, the Destination Unreachable of the ping packets are necessary.

>[!Question 4]
>What do the log items `IN, OUT, SRC, DST and PROTO` mean and why might they be useful?

A sample of LOG
```Bash

[ 1713.604741] iptables-dropped: IN=lo OUT= MAC=00:00:00:00:00:00:00:00:00:00:00:00:08:00 SRC=127.0.0.1 DST=127.0.0.1 LEN=84 TOS=0x00 PREC=0x00 TTL=64 ID=40263 PROTO=ICMP TYPE=0 CODE=0 ID=5 SEQ=113
[ 1714.628271] iptables-dropped: IN=lo OUT= MAC=00:00:00:00:00:00:00:00:00:00:00:00:08:00 SRC=127.0.0.1 DST=127.0.0.1 LEN=84 TOS=0x00 PREC=0x00 TTL=64 ID=40516 PROTO=ICMP TYPE=0 CODE=0 ID=5 SEQ=114
[ 1715.651754] iptables-dropped: IN=lo OUT= MAC=00:00:00:00:00:00:00:00:00:00:00:00:08:00 SRC=127.0.0.1 DST=127.0.0.1 LEN=84 TOS=0x00 PREC=0x00 TTL=64 ID=40720 PROTO=ICMP TYPE=0 CODE=0 ID=5 SEQ=115
```

These indicates show the information about the input and output network interfaces, the original destination and source of the packet. And last, the protocol, indicates whether it was tcp, udp or icmp (ping). It also provides information about which chain these packets went to (in this case, it is iptables-dropped).

>[!Question 5]
>When configuring a firewall, the order of the rules is of greatest importance! Why?

Because these packets will be processed as first-match rule. The `iptables` will handle rules as a linear list, once it finds appropriate rules, it immediately puts the packets to designated chains, even maybe that decision was wrong. For example, if the #1 rule is ACCEPT all, then #2 is to block some black listed IPs, so that, every packets will be safe regardless several came from malicious sources.

>[!Question 6]
>Consider a firewall which uses a default permit policy, and has no other rules set. Is the system secure? Is it useful?

First, not secure, totally like a normal cable, accept anythings, passes anything and allows everything. But in the normal case, it should have the least privileges principles, should not have totally open like this.

Useful in setup some virtual router, in testing, in sandbox environment, but too dangerous to use in real scenario.

>[! Question 7]
>Consider a firewall which uses a default deny policy, and has no other rules set. Is the system secure? Is it useful?

Secure, since nothing can pass. But not useful, like a black hole. Even in this scenario, if the default is applied for output and input with the loop back interface, the machine can not even talk to itself, so a total isolated environment.