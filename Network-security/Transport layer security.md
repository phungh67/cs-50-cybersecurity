Take away from last [[Network layer security]], IP cannot be relied to be a method for protecting the communication link, since almost all fields in the header can be abused (address, fragmented flags, time to live, and even the option fields).

# Scanning port - a reconnaissance step

>[! Challenge]
>Any machine has 65535 ports, and there are hundreds in a system. Take an example, for a /24 sub net, we have 254 machines. With a simple math, it takes 16 millions packets (16 and a half) to scan all of them. But we can also just scan common ports (80, 3306,...) - but of course, the system operators most likely configured to change these ports at the beginning.

A note to take is, there are 2 types of scanning: TCP and UDP. While the UDP is known for its speed (does not require the 3-ways handshake), scanning UDP ports is very hard, because:
- There is no mechanism to indicate that such a connection was made successfully (connectionless protocol).
- Take several packets to indicate a port's status. Since there is no way to distinguish if an UDP packet was dropped due to the firewall or due to the network problem (i.e no route)

There are 2 strategies with UDP scan:
- Send 0-bytes datagram to each port, does not interfere with application. If got ICMP port unreachable, host existed but port was closed. If no reply, it is really hard to say the packet was filtered by the firewall or not.
- Send a protocol-specific payload to common port, targeting specific service: DNS, SNMP, DHCP,...

## TCP scanning - based on IP header flags

There are several flags, used for controlling:
- URG: urgent - this packet will be prioritized to process first.
- ACK: acknowledge - this packet arrived successfully.
- PSH: push
- RST: reset - reset a connection (in a 3-ways handshake, delete that connection from Transmission Control Board).
- SYN: sync - begin a TCP connection with 3-ways handshake, successfully sending this packet means the server is up and accepts traffic in this port.
- FIN: finish - wrap up a session.

## `SYN` - a stealth scan

The `nmap` will send a `SYN` packet, when the server receives that packet, it will send back a response with `SYN/ACK` - indicates that "Port is open, go ahead and continue the communication process". 

In case the port is closed, a `RST` will be sent back.

But this scan can be detected under some tool, even wasn't logged, it still can be detect by `netstat` or `netcan` which will show that there were some traffics in state `established`.

## `ACK` - filtered scan

This scan is used to detect if a port was filtered or not.
Unlike SYN scan, it is not visible via `netcat` command. With an open port, always returns `RST`.

Can also be used to determine if the firewall is `stateless` or `stateful`. Since the `stateless` firewall allows `ACK` (only blocks `SYNs`).

## `SYN/ACK` technique

This is a combination of these two above methods with a purpose: to evade the firewall detection.
Normally, the `stateless` firewall will block `PING` (avoid ping-of-death and various attacks), but not `SYN/ACK` scan. 
Send the `SYN` to request entrance first, determine the status of port, if success, send `RST` to close the connection, avoid `netcat` and `netstat`.

## `FIN` scan

This scan is also stealthy, can not be observed by `netcat`. From `RFC 9293`, a traffic to a closed port should always returns `RST` AND, if an open port received any segment but not `SYN`, `ACK` or `RST` (those are essential to a "proper" communication session), the segment must be dropped. 

It takes the inspiration from `FIN` flag when a session was completed. So that if there is any response (`RST`), the scanned port is definitively closed, but if not, we are in.

**Note** in Windows, the `RFC 9298` was not followed strictly, so all the times, `RST` will be sent. But it is also helpful to get the OS footprint.

## Other types of scan:

- **Null** scan: does not set any flag at all - create an invalid message, that should have the same result as `FIN` scan.
- **Xmas** scan: set these flags on  `URG`, `PSH`, `FIN` response will be different based on how the `TCP` state machine in that system.

# Comparison between scans

## `SYN` scan
- `SYN` flag was set.
- `SYN/ACK` scans open ports quickly - indicate the openness, `RST` - closed and `filtered` = firewall protection.
- Best for port discovery.
## `ACK` and `SYN/ACK` scan
- `ACK` flag was set.
- No response = filtered, `RST` = live system existed,
- Best for checking firewall rule.

## `Xmas`, `Null` and `FIN` scan
- Combination many flags `PSH`, `URG`, `FIN` or zero.
- No response means open port, `RST` = closed port (only non - Window).
- Best for OS fingerprinting.

# `TCP` fingerprinting identifies and `OSes` scout.

This method focuses on detect the Operating system along with all the supported version of applications installed in it. Since most exploits relies on some vulnerabilities from specific version of some application, this attack is very useful and widely used.

**Passive** - inspect packet, take the characteristics from window size, time to live,... to determine.
**Active** send `TCP`, `IP` or `ICMP` messages and some malformed messages to port 0 - where the response is system - depended.


# Exploiting the `UDP`

Because `UDP` data does not have sequence number, abuse the packets is very easy. And also, it is impossible to detect on the `UDP` level.
Even the application could not detect the malformed data.
Furthermore, firewall and application are in separate layers. So that even the firewall encountered some malformed data, it maybe not be able to handle nor detect the modified "payload" (since the attacker tried to abuse the application level).

# Exploiting the `TCP`

## Zero size payload (Window size = 0)

As in the structure of the IP packet and also the scenarios that performed earlier with `nmap` and various scan strategies, the `IP` packet is extremely vulnerable. Especially the header.

This led to a famous attack (even today) `DOS` - Denial of Service.

**How-to**: the attacker tries to make the receiver's buffer full, upon that, it will send a packet of 0, signalling the sender to stop send any new packets (due to full capacity).

**Applied**: attacker created many fake connection, send a packet with `TCP` size = 0 and kept them alive. The sever was overwhelmed with these clients, but cannot send any data due to size = 0, resulted in legitimate connections could not be made nor served.

Even if the application closed the connection, it will be hanged in the state `FIN-WAIT` (wait for finish message). The impact is varies depends on the implementation of `DoS`.

## Small `IP` fragments

The header of `IP` packet should only be presented in the first fragment (if the message was split). The minimum value for the size is 20 bytes (IP) + 20 bytes (TCP) - so normally, for any message that is less than this amount, fragment is not necessary.

If a fragment was abused, that is, another piece of this message contains different information. The first fragment: port = 80, but the second, all the same, expected port = 1143.

Mismatched data could cause error in the system, so that, firewall should not allow short fragments (even with large message).
- Always drop the fragment that overwrote the packet information (mismatch with first piece).
- Since the `TCP` can figure out the `MTU`, so fragment is rare -> can drop them entirely.

## `TCP` flags

As seen in the last section about scanning, flags are versatile, indicating nature of messages, but can be abused to make attack.

- Performance algorithm indicates that three repeat `ACK` means "congestion control" - fast retransmit or slow start mode - affecting performance of the network.
- `PSH` - Push flag forced the message to be immediately delivered to application layer, causing mis-handling (since it may require all packets to begin processing).
- `URG` - urgent flag indicates that this packet should bypass the queue - triggering unexpected behavior of the system.

# Further attack

The attackers sometime don't simply want to mess up with the traffic or communication link, they also want a total control during a session.

###  Spoofing `TCP` segment requires many guessing

- Source, Destination and Service Port: guessing without challenging since these can be found more or less within previous communication session (email header, well-known ports,...).
- Source port, this is quite a challenge, both with "side" and "middle" since the communication port can be randomly assigned, several services are "fixed".
- `TCP` sequence number, this is the hardest part, since the number should be "legitimate" enough so that the forged message looks "okay" from the side of both server & client.
	- The initial sequence number `ISN` is easy to guess.
	- Also, the receiver's window makes the guessing easier (just need to land somewhere in-between is enough).
	- With a strong network bandwidth, exhaustive search can be performed quickly.

### Mitigation

- Need to match exactly the sequence number for the packet to be accepted.
- Authentication for the `TCP` (an option) and the session key will changed throughout the session (for each message) - using the `ID`.
- Also, to be more secure, the initial sequence number must be a little challenge `ISN` should be totally random and unique.


# Conclusion

>[! Take aways]
>**Port scanning** using some tools to discover if a port of a system is opened and accept traffic. Can use vary methods (depends on the purpose: detect, evade, or probe the firewall rules): `SYN`, `SYN/ACK`, `FIN`, `Null` and `Xmas`
>**Small fragment attack** - attack IP, but the abuse object is the header. Change the data in the header's field to confuse the firewall.
>**`UDP` attack** hard to detect even with both firewall and application combine together.
>**`TCP` packet injection** quite hard because the fake packet must be forged with many requirements, while source , destination and service port are really easy to get, the source port and the sequence number require much more effort to get correct result.
>**`TCP` session hijacking** keeps the server busy to deal with dummy clients so that no legitimate client can be severed. For example, send some zero payload packet.

