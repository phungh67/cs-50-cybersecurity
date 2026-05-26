# Link Layer Security

>[! Notes]
>Link layer is the layer that right beneath the Network layer (with IP and stuff). At this level, we know about MAC address, ARP (Address Resolution Protocol), DNS,... stuff

Currently, a secure infrastructure is implicity (that we hope it will happen, but cannot guarantee that it will always happen).

- `DNS` resolve hostname to IP properly (no hijacking, no tampering,...). Assume that `TLS` or `SSH` and Kerberos trusted on first use.

- Internet routing with `BGP` correctly forwards traffic to right destinations (no intervention or posioning).

- The link layer (ARP, DHCP) deliver data frames to the right host.

- The switchs are legitimate (firewalls, IDS,... operate correctly).

## Address Resolution Protocol

`ARP`- in charge of mapping IP address to the MAC (media access control) address (the physical address in the network interface, the physical network card).

- Designed with no security.
- Every node manages an ARP table, to store the information about MAC and IP relationship. Dynamic, short time to live, hours at switches and routers, but maybe minutes on OS.

The most dangerous, `ARP` cache can be poisoned, so that a malicious address can sneak into the table, hence altered all the information flow.

Some countermeasures exist:
- For ciritcal hosts (gateways, servers): static assign (some kind of hard code, no cache checking).

## DCHP assigns IP addresses and configuration parameters

DHCP stands for Dynamic Host Configuration Protocol, allow a clien on discovery, receives an IP address from DHCP server. The process is called DORA: Discover - Offer - Request and Accept.

Also lack of authentication.

Countermeasures:
- Only trust specific port on switches, another port - close imnmediately.
- Rate limiting to avoid attackers try to request and exhaust the IP pool.

## Existing defense methods

**DHCP Snooping**: switch keeps track of DHCP requests and relies:
- Binding table: IP - MAC - port - lease time.
- DHCP replies from untrusted port are dropped. (only allow on defined port).

**Dynamic ARP Inspection (DIA)**: checks all requests against DHCP binding table
- Can be host-based or network-based, used in combination with IDS.

**IP source guard**: drops packets without valid IP/MAC in the snooping table.

**Limit number of MAC addresses per port**:
- Limit the number of spoofed MAC.
- Also limit the number of fake MAC.

## MAC address flooding attack.

As mentioned before, link layer maybe one the most critical in the network. But since it operates very close with the physcial layer, so too much security means wasting computation powers and cost.

MAC, on the other hands, have one of the notable attack: flooding (same as the `SYN` flood in the layer 4).

**Switches** learn and store MAC addresses:
- Store in fast CAM (Content Addressable Memory): MAC - Port
- Separates traffic (performance and confidentiality).
- When memory is full, broadcast to all interfaces, same as when learning.

**MAC flooding**
- Constanly fill the table with garbage (fake/trashed MAC addresses), force traffic to be sent to all ports.


>[! Counter]
>There are two main ways to counter this attack but cannot guarantee 100% safety

**Port security**: enforce a policy to limit the number of MAC addresses per port. The said port can only learn first n addresses. Shut down if exceeded. Also configured timeout to refresh the table frequently.

**Port access control**: no traffic between protected ports. But they still can communicate with unprotected ports.

## MACsec - MAC security

Will encrypt and authenticate Ethernet frames hop-by-hop using GSM-AES.

But of course, requires supported hardware and every hop must support MACset.

## VLAN - an isolated method to mitgate the link layer attack

Only forwards frames to ports that belong to the same VLAN, for example:
- VLAN1 and VLAN2 in the switch S1.
- VLAN1 was designed to have port 1,3,4,5, while the rest are belong to VLAN2.
- S1 only forwards the frame (that has VLAN1 as destination) to port 1,3,4 or 5, leaving the other ports untouched.

Traffic within a VLAN stays in within that VLAN, including broadcast traffic.

This leads to isolation in the layer 2. Since Virtual is a logical method, if 2 VLANs want to talk with each other, must go through another router or layer 3 switch (where better security methods can be applied to inspect frames).

**Keys takeaway**: VLAN is not desgined for security, since it provides no cryptographic methods. It is only for isolating the broadcast domain, to limit the blast radius in case MAC spoofing or link layer attacks happen. Also, VLAN must rely on the correctly configured switches and other layer 2 components.

## DNS - common, but still no effective protection methods

### Issues

**Client just accepts** the first reply that matches the query.
- No cryptographic verification.
- A forged reply from attacker can win the race easily, depends on the location of the adversary on the communication channel.

**Man in the middle** easily happens in this case. Even the higher layers defense can verify the wrong parties (since no signature at all).

**DNS spoofing** can happen at every positions on the traversal path.

