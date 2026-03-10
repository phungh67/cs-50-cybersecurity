
# 1. Attack

The action of prevent or impair the authorized use of network, system or application by exhausting or terminating the resources such as CPU, memory, bandwidth and disk space.
There are two main types:
- Overload/flood attack to the network bandwidth, system resources or application resources.
- Crash and kill: find the vulnerabilities of the server, kill it.

In the form of C.I.A triad, which property this type of attack aims to? (A).

# 2. Differences between two types

The *Overload/Flooding* is conducted with these characteristics:
- From higher capacity network to the lower (so that is why we call it "flood").
- Causing the loss of traffic due to legally users are denied.
- A simple `ping` command with sufficient number and in relatively short time can cause the flood.
- ICMP flood, UPD flood and TCP Syn flood.
- Source of flood can be easily tracked down.

On the other hand, the *Crash and Kill* is a little different from that nasty fellow:
- Trigger bug in the system (poison packet).
- Classic examples:
	- Ping-of-death
	- Land attack

# 3. Flood attack

In the most naive understand, when a massive system tries to attack a smaller system, surely that one will fall, it is only the matter of time. Because the system's capacity is limited, it cannot response quick enough with a huge number of sending requests.

But luckily, in this scenario, the ID of attack is visible.

We have three main types (and we classify them based on the protocol):
- ICMP, uses the echo, ping request, typically allowed through.
- UDP flood, quickly because does not require handshake.
- TCP SYN flood, uses TCP packet and leaves the connection half-open.

>[! SYN attack]
> Although conducted by some thing related to the network (TCP packet, handshake,...) but actually, the target of this attack is not the network itself, but more about the memory. Because the system must retain an area within the memory to track all the open connection (to handle them, determine whether to close them or leaving them open). So because many slots were occupied, the legally request (from a client in the system) would be denied.

In this scenario, the attacker should:
- Use random addresses, that is unable to receive a feedback, so that the connection was left half-open,
- Need to strictly ignore the `RST` - reset command of the packet, because with that command, the connection will be terminated.


# 4. Distributed Denial of Service Attack (DDoS)

Well, to be short, it is a Denial-of-service attack that was conducted from many sources (the old method is just from a massive system, so it is easy to detect the root cause).
- Consists of multiple systems, allow a much higher traffic volumes to from a DDoS.
- It not only aims for single type of resource to consume like TCB, CPU,... it installs zombies into that server, forming a "botnet".

The control hierarchy:
- Attacker sits on top of this structure.
- Several handlers are controlled by the attacker machine.
- Handlers then execute agent (maybe infected earlier, some farm machines,...) to attack a single target.

Reflection attack and amplification attack:
- Use the normal behavior of the network.
- Try to send the packets with spoofed address to a normal server.
- Attacker try to send to that target (the one discover in the last step).
- Send many requests
- Used many protocols and many sources.
- Use the normal types of services to overwhelm the target.

# 5. Counter

The first method is to block these things, but well, since there are many ghostly addresses, so it is rarely implemented in the real scenario.

Rate limitation is also a very handy method to bound the maximum number of requests that an IP address can make always in a controllable range.

Modified TCP is also a good way to deal in this case.

>[! Response to Attacks]
>Identify the problem, the source by capturing and inspecting a packet. Also, we should take a step earlier in this scenario by designing a filter before accepting any incoming message. After analyzing these things, have ISP trace back to find the source, but since ISP is not internally, it takes time, and also requires legally permission to do this. But otherwise, have a contingency plan is a good idea to do so. After all, a good summarize and report are always needed.

