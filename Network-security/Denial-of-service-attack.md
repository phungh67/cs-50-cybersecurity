# Forewords

From the beginning of the Internet, the idea about a system that connects all machines, from the big (super computers, complex systems,...) to smaller one (personal workstations, some potato PCs,...).

Some facts (not to fact) about this type of attack:
- Launching a `DDoS` (distributed denial of serivce attack) is not too hard, in fact, you can hire a fleet of machines to do this just with 5$ per hour.
- This is an "active" attack, that is, easy to detect but very hard to completely mitigate it, so the goal is to lessen the impact and damage as much as possible. Thus, it already costs thousands if not billons of dollar to completely apply protection mechanisms.

Moreover, a down time, no matter how long, can affect severly to the owner's business.

# Early classification

In the category of `DoS` attack, there are several types to consider:
- Classic Denial of Service attack - aim to disrupt a service completely, making legitmate users cannot access the resources.
- Reflection and Amplification attacks - aim to make the damage larger, leading to uncontrollable damage and expand the blast radius.
- Distributed Denial of Service attack - evolution of the classic one, can be done by a mass number of machines, hard to trace the identity of attacker and root of the malicious payload.

# Definition and Categorization

>[! Definition]
>**Denial of Service - DoS** is an action that prevents or impairs the authorized users use the network, systems of applications by exhausting or terminating resource, including but not limiting to central processing unit (CPU), memory, network bandwidth and diskspace.

For short: Denial of Service is an attack that makes legitimate users cannot enter nor being servered by the system, causing chaos and damaging the user's experience.

**Classification - Categorization**
- Network bandwidth: overloading the network limit connecting the target to the Internet, for example: the Transmission Control Board (TCB), the number of `max_open_files`.

- System resource: symply enough, trying to consume as much as possible all resources from memory, disk space,... or even intentionally sending a malicious packet to trigger hidden bugs.

- Application resources: a new layer, it not simply take down the machine (since sometime it is quite challenging), so it just block the application from handling new request, for example, by filling all the allowed database connection, hence denies any new user's request.

>[! Example]
> Any flooding attacks will work if the attacker's machine is much more stronger than the victime. Some notable: `UDP` flood, `ICMP` flood, simply overwhelming the target with a bunch of packets. Posioned packets triggering hidden bugs,... And many more complex attacks that targeting application layer.

## `SYN` flood - a classic one, popular and easy to conduct.

The attacker will send as many `SYN` packets as possible to the victim. As in usual, the victim machine will send back `SYN/ACK` signal, wait for a final `ACK` to complete the three-way handshake.

But the thing is, the last message will never come becasue the attacke will modify the source IP address so that the victim will always wait for reply from "invalid" address, hence, keeping the connection open, consuming the slot in the Transmission Control Board.

Surely, the OS itself has mechanism to drop the connection after a while, but mostly, it takes 180 seconds (3 minutes) for the machine to drop the connection, and because attacker sent the messages at a faster rate than self-drop rate, the server will eventually trapped in the "full" state - denying any new connection (mostly from legitimate users).

**Mitihation strategy - RFC 4987**

**`SYN` cookies**

Short description, the server challenges the packet if it was from a flood before accepting it. Legitimate client will complete the challenge without any dificulities. while the flood could not do since the intention of attacker is never to complete the handshake.

Server sends $server_{ISN} = client_{ISN} + cookies(32 bits)$
Client sends $server_{ISN} + 1} back

Server then checks time (to determine if the packet was legitimate) and allocating memory for this session.

**Complications**

- Cannot send `TCP` options.
- A `stateless` firewall, as said, will block `SYN` packet, so the solution is to open the port that was flooded before.
- A `stateful` firewall, on the otherhand, does not rely on flags to control, surely vulnerable to the `SYN` packet, even with the cookie.

## Reflection attack - hide the attacker's identity

>[! Concept]
>Attacker sends `SYN` packets to a reflector with the victim's address as the source address. Then the `SYN/ACK` will be send to the victim, retransmit 5 times if no reply.

In this case, attacker's address was hiden perfectly, even with inspection, the admins could only find the address of a reflector - useless since it could be any machine in the network.

Since the reflector could be any machine, even internal machines (routers, DMZ,...) so cannot simply block these targets.

**Under-the-hood**

- When reflectors received a `SYN` packet, that means a communication request was initialized, it must send the `SYN/ACK` to complete the handshake. 
- Because the packet was forged with victim's address, the reflectors expected victim to reply with `ACK`. But since it never reply the reflectors will keep sending the packet again and again.
- On the other hand, the victim was overwhelmed with the record from reflectors and surely, these packets indeed consumed bandwidth. And if the victim tried to reply, x2 the consumption because it took resources to create repies.

## Amplification - in pair with Reflection attacke
>[! How]
>Simple, many reflectors can work together to take down a victim. And mostly the attacker will choose a strong enough machine to be the reflector.

Another type is a **Smurf** attack, with stolen IP address from the victim, it will send the broadcast packet to all other machines (broadcast of course - to all reachable machines in the network) to generate a `IMCP` ping flood with echo reply.

A similar attack calls **Fraggle** - use `UDP`-based service.

# Mitigation

There are many methods to mitigate, as mentioned before:
- Cookies for SYN.
- Perform rate limiting.
- Monitoring and whitelisting,...

One of many way is uRPF - Unicast Reverse Path Forwarding.

- Strict: if there is any packet came from an IP address, through any interface in this router, if there is a patch to reach that IP address, with the same interface, accept it, otherwise, it is a forged packet.
- Loose: as long as there is a way to reach the sender's IP address in the route table (IRT), accept it, otherwise, malformed packet.

