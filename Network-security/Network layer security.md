
>[! Foreword] 
>Network can be scan very fast, at a tremendous speed, under five minutes, a dedicated machine can know the whole IPv4 space. And today, many tools are published [[Lab 1 - Nmap and WireShark | Nmap and Wireshark]]. Even firewall could not properly defends our system from all of them [[Lab 2 - Firewall, go to classic with IP table | Firewall and IP tables]].
>

Even firewall devices are vulnerable, the proof is many Cisco or even Palo Alto Network firewall have many CVE that haven't been fixed yet.

To get a glimpse of how an attack can be conducted, let's take a look at the IP header. And even inside an IP packet, all fields can be attacked.

![[ip-field-illustration.png]]

In the above figure, the IP header is at least 20 bytes long, or, at most 20 bytes long. Hence, a packet should be larger than 20 bytes (since it can be an empty packet, but still must have the appropriate fields to determine the source, destination,...).

So if there is any packet that is less than 20 bytes, we know these packets were malformed.

# IP address spoofing

>[! Definition]
>This attack focuses on forging an artificial packet, to act as the legitimate packet, hence, steal the communication line between the client and server.

To forge a packet with stolen IP address is easy, we have the source and the destination, just need to make sure these 2 arguments are correct.

But to get a reply to the said forge packet, it is much more difficult because there is RFC 2827 that only allows outgoing packets with IP address belong to the system to be sent.

This attack has 1 variant names LAND attack - create a packet with both source and destination are the same as the host's IP. The goal is to trigger the hidden bugs in the implementation layers, hence crash and take down the system.

The take away from the IP spoofing:
+ Sender anonymity: attacker cannot be identified (they just need to hide the actual IP by forging something).
+ Easily to blame someone for sending these packets (because IP address can be faked).
+ Can exploit trust between hosts (since probably there is no additional security check between trusted entities).

The length field, as said before, can be exploited. If the actual length of the datagram is larger than the length field in the header, so there is maybe a buffer overrun , very dangerous, but if less than it, maybe the packet will contain old contents (also dangerous). So that, always perform the boundary checking to avoid error.

# IP fragmentation

When a packet is to large for the Maximum transmission unit (MTU), it must be fragmented. There are three flags in the header for this case:
+ Fragment ID - specify the ID of that packet, all pieces must have the same ID, indicate that they are belong to one and only packet.
+ MF flag - indicate that there are more fragment pieces after this packet.
+ Offset - indicate the position of this packet in the assembled datagram at the end.

The adversaries aim to trigger the reassembly bugs:
- Teardrop or bonk attack.
- With teardrop, they sent two identical and overlapped fragments. First fragment can sit perfectly in the previous, hence resulted in a negative of the number of bytes to copy, then that unsigned integer become very large positive number, overwhelmed the system. And the second fragment overlapped partially with the last legitimate fragment, so the reassembly became ambiguous.

# Idle scanning

The goal of this attack is to learn if the system's ports are opened for some specific systems (trusted entities).

First, need to reconnaissance - send the `SYN` to the zombie, to learn the fragment ID counter of the zombie, then use that information to forge a new `SYN` packet.

If the zombie is a trusted one, it will receive an unwanted `SYN/ACK`, else send `RST`

If the zombie is trusted, successfully received reply from sever, the ID will be increased, then within attack scenario, increase ID = success

# ICMP message 

Internet Control Message Protocol - ICMP is familiar in the usage with the command `ping`to check if any host is currently up or not.

In the collaboration with `traceroute`the ping message has another field calls Time To Live TTL - to avoid loop and circular sending in a broken network.

ICMP can be used to test a firewall is stateless or stateful. If send with stateless, it can be replied, otherwise, stateful will drop and return the time exceeded message.

![[Pasted image 20260516161131.png]]

Should allow the type 3 code 4 for MTU discovery path. Otherwise the sender will be confused because it does not know if the message was dropped by firewall blocked or because of exceed size.