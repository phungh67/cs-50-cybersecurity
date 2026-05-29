
>[! Definition]
>Any devices, any machines, specialized machines, multiple machines,... in combination to inspect, filter, defense the network against any malicious payloads, packets, we can call it `firewall`.

# A brief history of firewall

- Routers preventing own problems to spread (segmentation).
- Routers with statically packet filtering - early concept of `stateless` to remove unwanted traffic (no memory at all, only match and action).
- Bastion host and proxies - a specialized host machine taking care of a few necessary services. May support user authentication.
- `Stateful` packet inspection - not simply match and drop/pass, packet was inspected carefully with sophisticated techniques and states were kept persistent.
- Application layer inspection - same concept but in application layer. Different modules for different protocol.
- Host-level firewall: `netfilter` and `iptables`.

# Proxies and Gateways

>[! Definition]
>Proxy - a way to redirect the request so that the client did not contact directly with the server. Instead, a middle server will involve, handling that request and pass it to the destination.

All headers are removed, the proxy will create a new header to indicate that these packets were handled by proxy earlier, the server just needs to focus on creating responses.

(reading for headers abuse [[Network layer security]])

Gateway and proxy are 2 terms, but refer to the same concept: intercept incoming or outcoming traffic - not allow the client and server communicate directly.

# Types of firewall

## Static packet filter firewalls - layer 3 (network layer)

- `Stateless` - does not understand `TCP`.
- Just compare the IP header (source, destination,...) with an access control list.
- Can detect IP spoofing (compare sequence number).
- Fast and cheap, usually a part of a router.
- A small component in comparison with a real router

## Dynamic (self-modifying) packet filter (layer 4 - Transport)

- Rules can be changed dynamically by the filter itself to understand `SYN-SYN/ACK-ACK` sequence.
- Still a filter (need some access control list to fully functional).

## `Stateful` packet inspection (layer 4 and higher)

- Access Control List consulted at connection establishment; a stable table is used for established connections.
- State aware, understand about `TCP` connections and states.
- Can also inspect application level packets.

## Circuit-level gateway (generic proxy) (layer 4)

- Replay `TCP` connection; unpacks then repacks again for other side.
- Hide internal structure.
- May perform authentication.

## Application level proxy - gateway

- Designed for one application (SSH, FTP,...).
- Unpacks application data and repacks it on the other side.
- May perform authentication.

## `Stateful` application level packet inspection

- Inspect several application layer protocols.
- Can have signatures for known attack patterns, detect ping-of-death or some non standard commands in FTP,...
- May require special hardware for performance.

## Air-gap firewall - on special purposes.


# Screening routers - normally static packet filter with `stateless`

>[! Concept]
>Based on access control list and simple match-and-execute pattern, for example `If source == 10.0.10.0/24 and port == 25 block`.

The filter is based on:
- Source and destination of the traffic.
- The incoming port.
- Interface that the packet arrived.

Typically implemented at the border routers (avoid overclocked the main border router - firewall). Very good at filter ingress and egress traffic, especially for the high-frequency but low-complexity attack (i.e, only target with IP address, port and interface, naive scanning, amateurs ...).

>[! Important notes]
>About Ingress and Egress filter, there are several recommendations that should be obeyed

### Private address `CIDR` blocks should not be forwarded by border routers - they must communicate securely within `subnet` scope

"Otherwise, there is a high chance that intruders are pretending that they are internal communicators." Hence, these blocks must be dropped:
`10.*.*.*`
`172.16.*.*` - `172.31.*.*` 
`192.168.*.*`
`169.254.*.*`

### Broadcast, multicast, reserved addresses should be banned.

### Malformed packets should be dropped.

Check [[Transport layer security]], packet with flag `FIN`and `SYN` should be drop, also tiny fragment.

## Ingress
### Only allow specific type of `ICMP` messages

Type 0 - echo reply (to probe the network).
Type 3 - destination reached, fragmented needed - for `MTU` discovery
Type 4 - Drop
Others - Drop

Pass all other (for the main border firewall)

## Egress

### Only allow specific type of `ICMP` messages

Type 8 - echo ping
Type 3 - port unreachable response - block to avoid port scanning
Type 11 - TTL exceeded - prevent remote scanning (and hierarchy discovery)
Others - drop

### Drop `TCP` some port (some blacklisted)

# The main border firewall

(Comes after screening router firewall) predominantly uses `stateful` packet inspection filtering techniques.

This firewall keeps a table for state (from where to where, what was sent and current sequence number,...).

Once a legitimate packet was replied successfully, the firewall would update its state table to reflect connection's status correctly.

Most firewalls allow interfaces in Access Control Lists. For example, each firewall typically has 8 ports, each corresponds to one interface.

```Bash
Proto | Interface | IP addr       | Port
TCP   | 1 (ext)   | 60.55.33.12   | 42013
...
TCP   | 8 (dmz)   | 10.5.1.5      | 443
```

With `UDP` packet, should keep a property names "timeout" to avoid `UDP` scanning.

>[! Firewall principles]
>1. Check the connection state table first.
>2. If not found, check the access control list (to allow or not).
>3. If not okay, pass the packet and update the state into the connection table (for easy future lookup).

Rule table in the firewall is inspected as top-down:
- First match, done and leave the table, so the order of rules is very very extremely important.
- Last rule in the table (at least the main border) should always `DROP ALL` (since all necessary rules were matched earlier, rest - drop).

Several additional steps to inspect these packets at application level should be added. But with some notes:
- Increase processing time due to multi layers were applied.
- Maybe dedicated/specialized hardware is needed.

## Problem with FTP protocol

>[! Problems]
>Since this protocol tries to open new ports, so the rule may not keep up with the changes speed of these newly come ports.

Because the process of FTP will look like:
- A request was made at port 21 - the control port of this protocol.
- With each file, a new port `p` will be opened to transfer this file through it.

Also, the inspection for these packets should be done on application level because maybe viruses were hidden within files.

## SPI firewall - Short summary

This is the dominated technique for Main Border Firewall.

Security:
- Can handle almost all type of flows `TCP`, `UDP`, `ICMP`.
- Best choice if one firewall is used.
- Can do some application filtering: `FTP`, `HTTP`, `DNS`,...

Simplicity:
- Rules could be complex.
- Checking outgoing traffic is fast and easy.
- The processing time increases as the number of rules increase.

`Statefull` inspection:
- Cost effective - since many firewalls are run in standard operation systems.
- Hardened system with special packet handling software.


# Network Address Translation `NAT` - an alternative for firewall solution

>[! Notes]
>Internal IP address will be translated into one (or more) official address, with also a different port number - one per connection and only if necessary.

A company will have a limited number of public IP(s) but maybe hundreds of internal machines. For each outgoing connection to the Internet, a mapping (source IP - Port number) with (port number - Public IP) was made.

Clients can request `NAT` device to open ports with Port control protocol. `NAT` gateway stores several information:
- With `TCP` and `UDP`: source and destination ports and addresses.
- `ICMP` also source and destination, additionally query ID. Also, the echo reply information is modified by the `NAT` one again.

`NAT` also requires session termination handling - cannot wait forever for a packet, since it must handle many connections - need to reserve spots.

>[! Final takeaways]
>Firewall can be deployed at almost all layers (in the model of OSI). The only layer that was left out is physical layer, with cable, with switch, with plug-in-and-out stuff.
>So, at least, it protect [[Network layer security]], [[Transport layer security]]

And, a firewall cannot switch its natural, from passive match-and-drop (`stateless`) to dynamically store - inspection and decide `stateful` in any means.

Also, Denial of Service [[Cybersecurity/cs-50-cybersecurity/Network-security/Denial of service attack|Denial of service attack]] is a kind of attack that we can only mitigate, cannot avoid 100% percent since all machines and systems have their limits (max open files, CPU power, memory capacity,...).

If a firewall was overloaded with traffic, it must drop all traffic, uninspected, this somehow achieved the goal of Denial of Service attack.

