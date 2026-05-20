
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